# Course Structure

## Purpose

Определяет структуру последовательного практического mid+ курса по администрированию Linux:
порядок модулей, обязательный состав каждого модуля, язык материалов и требования к безопасной лаборатории.

## Requirements

### Requirement: Sequential modules

Курс SHALL быть организован как упорядоченная последовательность модулей. Обучающийся MUST проходить модули в порядке syllabus: 01–11, затем 13–15, затем 16–20, затем капстоун 12.

#### Scenario: Learner opens syllabus

- **GIVEN** обучающийся открыл `course/syllabus.md`
- **WHEN** он выбирает следующий шаг
- **THEN** видит модули 01→11, затем 13→15, затем 16→20, затем 12, и зависимости между этапами

### Requirement: Mid-plus scope

Курс SHALL позиционироваться как mid+ с эшелоном после MVP: сжатый bootstrap 01–04; обязательные блоки TLS, MAC, metrics/alerts, Ansible IaC и off-host DR; затем модули 16–20 (perf, identity, HA, containers, advanced net) до капстоуна.

#### Scenario: Capstone prerequisites

- **GIVEN** обучающийся готов сдавать модуль 12
- **WHEN** сверяется с syllabus
- **THEN** видит зависимости от модулей 13–20 и must-критерии TLS, metrics, Ansible, DR, postmortem, плюс усиление HA или identity или containers

### Requirement: Post-MVP echelon modules

Курс SHALL включать модули 16–20 с полным набором файлов модуля и практическими лабораторными (не только теория): performance-capacity, identity-sssd, ha-reliability, containers-orchestration, advanced-networking.

#### Scenario: Echelon module exists

- **GIVEN** каталог `modules/16-performance-capacity/` (и аналоги 17–20)
- **WHEN** модуль считается готовым
- **THEN** в нём есть `README.md`, `theory.md`, `lab.md` и `checklist.md` на русском с циклом Inspect→Change→Verify→Document→Automate

### Requirement: Module contents

Каждый модуль SHALL содержать: краткую теорию, лабораторную работу с заданиями, критерии приёмки и чеклист самопроверки.

#### Scenario: Module folder layout

- **GIVEN** каталог `modules/NN-slug/`
- **WHEN** модуль считается готовым
- **THEN** в нём есть `README.md`, `theory.md`, `lab.md` и `checklist.md`

### Requirement: Russian language

Весь учебный контент курса SHALL быть на русском языке.

#### Scenario: Content language

- **GIVEN** любой файл в `course/` или `modules/`
- **WHEN** его читает обучающийся
- **THEN** пояснения, задания и критерии приёмки написаны по-русски

### Requirement: Lab safety

Курс SHALL явно требовать практику только в изолированной лаборатории (ВМ, контейнер или выделенный стенд).

#### Scenario: Before first lab

- **GIVEN** обучающийся читает `course/lab-setup.md`
- **WHEN** готовит окружение
- **THEN** получает требования к ВМ, сеть, снимки и запрет на работу на продакшен-хостах

### Requirement: Dual-distro coverage

Там, где команды Debian/Ubuntu и Rocky/Alma расходятся, материалы SHALL давать обе ветки явно.

#### Scenario: Package or firewall divergence

- **GIVEN** лабораторное задание затрагивает пакетный менеджер, firewall или MAC
- **WHEN** обучающийся читает `lab.md` / `theory.md`
- **THEN** видит таблицу или параллельные команды для обеих семей
