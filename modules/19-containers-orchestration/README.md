# Модуль 19 — Контейнеры и оркестрация lite

**Цель:** поднять контейнер на хосте (Podman или Docker: run/volume/publish + systemd/quadlet или compose) и **k8s lite**: Deploy+Service+probes и один рабочий манифест на kind/k3s/minikube.

**Время:** 4–6 часов  
**Зависит от:** 06, 07, **15** (сначала Ansible, потом оркестрация)  
**Планка:** не подготовка к CKA; одного кластера на одной ВМ достаточно.

| Файл | Содержание |
|------|------------|
| [theory.md](theory.md) | контейнер vs ВМ, probes, Deploy/Service |
| [lab.md](lab.md) | Podman/Docker + манифест на kind/k3s |
| [checklist.md](checklist.md) | Самопроверка |
