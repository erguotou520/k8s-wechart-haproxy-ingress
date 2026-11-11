# k8s-wechart-haproxy-ingress

k8s-1-25-wechart 集群（DO SGP1, K8s 1.34）的 HAProxy Ingress Controller 安装清单，
由 ArgoCD Application `haproxy-ingress`（ns `argocd`）管理，同步到集群。

## 内容

| 目录/文件 | 说明 |
| --- | --- |
| `crds/` | 10 个 CustomResourceDefinition（`ingress.v1/v3.haproxy.org`），版本与控制器一致（v3.2.19） |
| `00-namespace.yaml` | Namespace `haproxy-controller` |
| `10-rbac.yaml` | ServiceAccount / ClusterRole / ClusterRoleBinding（与上游官方 manifest 一致） |
| `20-ingressclass.yaml` | IngressClass `haproxy`（controller `haproxy.org/ingress-controller`） |
| `30-configmap.yaml` | 全局 ConfigMap（超时、ssl-redirect、XFF、Frontend CR 挂载），nginx 行为对齐 |
| `40-deployment.yaml` | 控制器 Deployment，镜像固定 `haproxytech/kubernetes-ingress:3.2.19` |
| `50-service.yaml` | NodePort Service（31080/80 → 8080，31443/443 → 8443） |
| `60-frontend-http.yaml` / `70-frontend-https.yaml` | 两个 Frontend CR：body 三档限制（1m/16m/无限制）、XFF 头对齐、HSTS |

## 来源与版本

- 上游：<https://github.com/haproxytech/kubernetes-ingress> tag `v3.2.19`
- `crds/` 直接取自上游 `crs/definition/`（v3.2.19），未做修改
- `10-rbac.yaml`、`40-deployment.yaml` 以官方 `deploy/haproxy-ingress.yaml`（v3.2.19）为基准，
  仅改动：镜像打 tag `:3.2.19`（官方不打 tag，默认 latest）、增加 readiness 探针
- 可行性调研与注解映射报告：`docs/haproxy-ingress-migration-feasibility.md`（wechart 工作区）

## 当前状态（2026-11-11）

Phase 1：仅安装控制器资源，**不承接流量**。
- 现有 43 个 Ingress 仍由 ingress-nginx（IngressClass `nginx`）处理，经 DO LB 188.166.204.246
- 后续 Phase 2 再创建新的 DO LB 指向本 Service 的 NodePort，然后按域名灰度切换
