# mlops-pm-analysis

PM-аналіз результатів ML-експериментів у MLflow і Prometheus: чи можна передавати модель у staging.

## Задача

Класифікація ірисів (датасет Iris, 150 квіток, 3 види). Сплітування 70/30, тобто в test 45 квіток. У MLflow є експеримент `iris-classification` з 5 runs: дві логістичні регресії, дерево рішень глибини 1 і два random forest. Метрики: accuracy і log loss.

## Використані дані

| Файл | Що на ньому |
|---|---|
| `screenshots/input_data.pdf` | Опис задачі та вхідних даних від ментора |
| `screenshots/mlflow_runs.png` | Таблиця runs з параметрами й метриками |
| `screenshots/mlflow_runs_charts.png` | Порівняльні графіки accuracy і loss у MLflow |
| `screenshots/mlflow_best_run_rf-depth3.png` | Сторінка кандидата: параметри, метрики, run ID, теги |
| `screenshots/mlflow_artifacts.png` | Артефакти rf-depth3 (збережена модель) |
| `screenshots/mlflow_artifacts_rf-small_empty.png` | rf-small: артефактів немає |
| `screenshots/grafana_metrics.png` | Графік `mlflow_accuracy` у Prometheus |
| `screenshots/prometheus_loss_graph.png` | Графік `mlflow_loss` у Prometheus |
| `screenshots/prometheus_accuracy_table.png`, `screenshots/prometheus_loss_table.png` | Табличні значення метрик у Prometheus |

Скриншоти метрик зроблені в Prometheus UI, а не в Grafana. Метрики ті самі, з мітками `run_id`, `run_name` і `model_type`.

## Кандидат

**rf-depth3**: random forest, 100 дерев, глибина 3, run ID `2ddc4f9d563c43e5a7c6c8523a580db3`.

- accuracy 0.956 проти 0.933 у baseline;
- loss 0.141 проти 0.191 у baseline;
- єдиний run зі збереженою моделлю.

## PM-рішення

**Need more experiments.** Кандидат найкращий, але за accuracy перевага лише одна квітка з 45 на одному спліті. До того ж run не можна повністю відтворити: Source порожній, немає версії даних. Також немає метрик по класах, порогів і rollback-плану.

## Ризики

- Перевага кандидата може бути випадковою: маленький test, один спліт, без cross-validation.
- Модель обирали лише за accuracy і loss, без recall по класах і confusion matrix.
- Немає артефактів для baseline та інших runs, тож нема на що відкотитись.
- Невідомі код і версія даних для rf-depth3, run запущений під `root`.
- Немає погоджених порогів, owner і моніторингу після деплою (drift).

Повний risk register у `pm_report.md`, розділ 6.

## Що зробити перед production

1. Cross-validation для baseline і rf-depth3, логувати модель у кожному run.
2. Додати recall по класах і confusion matrix.
3. Логувати git commit і версію датасету, запускати runs під персональними акаунтами.
4. Погодити пороги для staging і production.
5. Зареєструвати модель у Model Registry і мати попередню версію для rollback.
6. Налаштувати моніторинг після деплою і призначити owner моделі.
7. Визначити, хто відповідає за витрати, сховище артефактів, retention метрик і cleanup стенду.
