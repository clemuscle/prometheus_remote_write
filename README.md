# Prometheus remote write

Bac à sable pour tester le *remote write* de Prometheus : plusieurs Prometheus
d'environnement envoient leurs métriques vers un Prometheus central.

```
prometheus_env1 (:9091) ──┐
                          ├── remote write ──> prometheus_monitoring (:9090) ──> Grafana (:3000)
prometheus_env2 (:9092) ──┘
```

Chaque environnement ajoute le label `environment` (`env1`, `env2`) pour
distinguer les séries côté central.

## Contenu

| Dossier | Rôle |
|---|---|
| `prometheus_docker_compose/` | Démo locale avec Docker Compose : 2 Prometheus d'environnement, 1 Prometheus central (récepteur remote write), Grafana |
| `prometheus_cluster_aks/` | Terraform pour un cluster AKS avec Prometheus et Grafana déployés dessus |

## Démo locale

```bash
cd prometheus_docker_compose
docker compose up -d
```

- Prometheus central : http://localhost:9090 (tester par exemple `up{environment="env1"}`)
- Prometheus env1 / env2 : http://localhost:9091, http://localhost:9092
- Grafana : http://localhost:3000 (admin / admin), source de données `http://prometheus_monitoring:9090`

Le récepteur est activé sur le Prometheus central par `--web.enable-remote-write-receiver`.

## AKS

```bash
cd prometheus_cluster_aks
az login
terraform init && terraform apply
```

Crée un groupe de ressources, un cluster AKS de 3 nœuds et une clé SSH, puis applique
les manifestes de `config/` (Prometheus et Grafana dans le namespace `monitoring`).
Penser à `terraform destroy` après usage.

À compléter : le provider `kubernetes` et le namespace `monitoring` ne sont pas encore
déclarés, et le remote write n'est pas encore configuré sur cette version.
