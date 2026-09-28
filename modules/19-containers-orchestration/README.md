# Модуль 19 — Контейнеры и оркестрация lite

**Цель:** Podman или Docker на хосте (run/volume/publish + systemd/quadlet или compose) и **k8s lite**: Deploy+Service+probes + один рабочий манифест на kind/k3s/minikube.

**Время:** 4–6 часов  
**Зависит от:** 06, 07, **15** (IaC-мышление; оркестрация после Ansible)  
**Планка:** не CKA; один кластер на 1 ВМ достаточен.

| Файл | Содержание |
|------|------------|
| [theory.md](theory.md) | контейнер vs ВМ, probes, Deploy/Service |
| [lab.md](lab.md) | Podman/Docker + kind/k3s манифест |
| [checklist.md](checklist.md) | Самопроверка |
