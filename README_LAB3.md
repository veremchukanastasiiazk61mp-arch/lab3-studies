# ЛР3. Сервіс у Kubernetes (варіант 5: k3d, R = 5, 500m/1 CPU, HPA 70 %, стрибок 2,5·X(1), Д6)

## Що потрібно встановити (Windows)
- Docker Desktop (вже є), Git Bash (вже є)
- `kubectl` і `k3d`: у PowerShell `winget install Kubernetes.kubectl` і `winget install k3d.k3d` (або `choco install kubernetes-cli k3d`), потім закрити й відкрити Git Bash
- перевірка: `kubectl version --client` і `k3d version`

## Частина 1 (кроки 1-5): кластер, база, сервіс, моніторинг
```
cp .env.example .env          # змініть пароль
scripts/cluster.sh up         # створює кластер, збирає і завантажує образ, розгортає все
scripts/verify_part1.sh       # перевірки кроків 3-5 (з них робляться скріншоти)
scripts/cluster.sh down       # повне видалення кластера
```
Перша збірка займає кілька хвилин (скачування образів k3s, postgres, prometheus).

## Частина 2 (кроки 6–7): генератор і серія без автомасштабування
Генератор (`loadgen/stepload.py`) за кроком 6: 8·R = 40 постійних зʼєднань, закриття зʼєднання після 100 запитів,
сходинка 30 с, середнє за останні 20 с. Модель однакова в усіх прогонах.

```
pip install psutil                       # необовʼязково: завантаження хоста під час прогонів
python scripts/run_series_k8s.py         # N = 1, 2, 3, 4, 5 по черзі (орієнтовно 40-60 хв)
```
Результати: `results/step_nN.json`, `results/step_nN_seconds.csv`, `results/prom_nN.json`, `results/manifest.json`.
Повтор серії (щоб оцінити розкид): `python scripts/run_series_k8s.py --tag rep2_`.

Під час прогонів не користуйтесь комп'ютером і закрийте браузер та інші програми.

## Частина 3 (кроки 10–12, Д6): стрибок навантаження без автомасштабування і з HPA
Профіль: 30 с на 0,3·X(1), 60 с на рівні стрибка, 90 с на 0,3·X(1); рівень стрибка = min(2,5·X(1), 0,9·X(R)).
```
python scripts/run_profile_k8s.py --mode fixed                  # крок 10: 5 реплік без HPA
python scripts/run_profile_k8s.py --mode hpa                    # крок 11: HPA, ціль 70 %, вікно 30 с
python scripts/run_profile_k8s.py --mode hpa --init-delay 15    # Д6: початкова ініціалізація 15 с
python analysis/analyze_elasticity.py                           # метрики еластичності, діаграми
```
Результати: `results/profile_*`; таблиці `results/analysis_elasticity.md`, діаграми `report/lab3/profile_*.png`.
