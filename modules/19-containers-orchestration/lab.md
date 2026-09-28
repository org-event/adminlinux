# Лаборатория 19 — Podman/Docker + k8s lite

## Цель

Запустить контейнерный веб с volume и publish, оформить автостарт (systemd/quadlet или compose), поднять kind **или** k3s **или** minikube и применить один манифест Deployment+Service с probes.

## Окружение

- `srv` с RAM ≥2 ГБ (для k3s/kind комфортнее ≥3–4 ГБ)
- `cli` для проверки NodePort/порта снаружи
- Снимок `before-containers`
- Firewall: опубликуйте только lab-порты

Рекомендация: **Podman + k3s** на `srv` *или* **Docker + kind** — что легче поставить в вашей семье.

## Задания

### 1. Inspect

```bash
which podman docker nerdctl 2>/dev/null
systemctl is-active docker podman.socket 2>/dev/null || true
free -h; df -h /
```

Выберите runtime и зафиксируйте в `19-stack.md`.

### 2. Change — контейнер на хосте

1. Запустите учебный HTTP (например `nginx` или `ghcr.io/...` лёгкий image):
   - `-p 8080:80`
   - volume с кастомным `index.html` → `/usr/share/nginx/html` (или аналог)
2. Verify: `curl -s http://127.0.0.1:8080/` и с `cli` на IP `srv:8080`.
3. Остановка/удаление контейнера **не** должна стереть volume-данные — проверьте.

Evidence: `19-container-run.txt`.

### 3. Change — автостарт

Один вариант:

- **Quadlet** / systemd unit `podman generate systemd` → enable --now; **или**
- `compose.yml` + systemd unit `docker compose up -d` / `podman-compose`.

Reboot *или* `systemctl restart <unit>` → снова `curl` 200.

### 4. Change — k8s lite cluster

| Выбор | Короткий путь |
|-------|----------------|
| k3s | install script на `srv`; `kubectl` через `/etc/rancher/k3s/k3s.yaml` |
| kind | `kind create cluster`; kubectl context kind |
| minikube | `minikube start --driver=docker|podman` |

`kubectl get nodes` → Ready. Evidence: `19-nodes.txt`.

### 5. Change — манифест Deploy+Service+probes

Один файл `~/lab-notes/19-app.yaml` (или два):

- Deployment: 1–2 replicas, image nginx (или ваш);
- containerPort 80;
- **readinessProbe** и **livenessProbe** (`httpGet` `/` или `tcpSocket`);
- Service: NodePort или ClusterIP+port-forward для проверки.

```bash
kubectl apply -f 19-app.yaml
kubectl rollout status deploy/...
kubectl get pods,svc -o wide
```

Verify:

- `kubectl describe pod` показывает probes;
- доступ с `cli` через NodePort **или** `kubectl port-forward` + curl (зафиксируйте способ).

Негатив (кратко): сломайте readiness (неверный path) → Endpoints пустые / не Ready — затем почините.

### 6. Automate

`lab-k8s-smoke.sh`: `kubectl get deploy` Available; `curl` к сервису; exit 1 иначе.

### 7. Document

Что на хосте vs в кластере; куда делись логи (`kubectl logs` / journal); ресурсные лимиты как долг; почему это не прод-HA.

## Критерии приёмки

- [ ] Контейнер: run + volume + publish проверены с cli
- [ ] Автостарт через systemd/quadlet/compose
- [ ] Кластер kind/k3s/minikube: node Ready
- [ ] Манифест Deployment+Service+probes применён и отвечает
- [ ] Негатив по readiness (или эквивалент) зафиксирован
- [ ] Smoke-скрипт зелёный

## Подсказки

| Семья | Заметки |
|-------|---------|
| Debian/Ubuntu | `podman` из репо / Docker CE; k3s хорошо ставится на чистый srv |
| Rocky/Alma | `dnf install podman`; SELinux labels для volume (`:Z`/`:z` в Podman) |

Не публикуйте Kubernetes API в интернет. kind внутри Docker — не вложенный тяжёлый prod.

## Очистка

Можно оставить кластер для капстоуна или `k3s-uninstall.sh` / `kind delete cluster`. Volume lab удалите, если место кончилось.
