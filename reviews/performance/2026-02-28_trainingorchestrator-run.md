# TrainingOrchestrator.run

**Type:** Performance Analysis
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/ai-threat-detection/backend/ml/orchestrator.py
**Language:** python
**Lines:** 73-173
**Complexity:** 4.0

---

## Source Code

```python
def run(
        self,
        X: np.ndarray,
        y: np.ndarray,
    ) -> TrainingResult:
        """
        Execute the full training pipeline
        """
        self._output_dir.mkdir(parents=True, exist_ok=True)

        split = prepare_training_data(X, y)

        logger.info(
            "Split: train=%d val=%d test=%d normal_train=%d",
            len(split.X_train),
            len(split.X_val),
            len(split.X_test),
            len(split.X_normal_train),
        )

        with VigilExperiment(self._experiment_name) as experiment:
            experiment.log_params({
                "epochs": self._epochs,
                "batch_size": self._batch_size,
                "n_samples": len(X),
                "n_attack": int(np.sum(y == 1)),
                "n_normal": int(np.sum(y == 0)),
                "n_features": X.shape[1],
            })

            ae_result = self._train_ae(split.X_normal_train)
            ae_metrics = {
                "ae_threshold": ae_result["threshold"],
                "ae_final_train_loss": ae_result["history"]["train_loss"][-1],
                "ae_final_val_loss": ae_result["history"]["val_loss"][-1],
            }

            rf_result = self._train_rf(split.X_train, split.y_train)
            rf_metrics = rf_result["metrics"]

            if_result = self._train_if(split.X_normal_train)
            if_metrics = if_result["metrics"]

            self._export_models(ae_result, rf_result, if_result)

            experiment.log_metrics(ae_metrics)
            experiment.log_metrics({
                f"rf_{k}": v
                for k, v in rf_metrics.items()
            })

            try:
                ensemble = validate_ensemble(
                    self._output_dir,
                    split.X_test,
                    split.y_test,
                )
                experiment.log_metrics({
                    "ensemble_precision": ensemble.precision,
                    "ensemble_recall": ensemble.recall,
                    "ensemble_f1": ensemble.f1,
                    "ensemble_pr_auc": ensemble.pr_auc,
                    "ensemble_roc_auc": ensemble.roc_auc,
                })
                passed = ensemble.passed_gates
            except Exception as exc:
                logger.exception("Ensemble validation failed")
                print(
                    f"  WARNING: validation raised"
                    f" {type(exc).__name__}: {exc}",
                    file=sys.stderr,
                )
                ensemble = None
                passed = False

            for name in (
                AE_FILENAME,
                RF_FILENAME,
                IF_FILENAME,
                SCALER_FILENAME,
                THRESHOLD_FILENAME,
            ):
                experiment.log_artifact(self._output_dir / name)

            run_id = experiment.run_id

        logger.info(
            "Training complete: passed_gates=%s run_id=%s",
            passed,
            run_id,
```

---

## Performance Analysis

### Performance Analysis

#### Time Complexity
The time complexity of the `run` method is primarily determined by the training processes (`_train_ae`, `_train_rf`, `_train_if`) and ensemble validation, which are likely to be O(n * m) where n is the number of samples and m is the number of features. The logging operations and artifact exports contribute minimally.

#### Space Complexity
The space complexity is moderate due to storing training results (`ae_result`, `rf_result`, `if_result`) in memory. Ensuring that these objects are properly managed can help avoid unnecessary memory usage.

#### Bottlenecks or Inefficiencies
1. **Redundant Logging**: The logging of split sizes and parameters could be moved outside the experiment context to reduce overhead.
2. **Exception Handling**: The exception handling for ensemble validation is resource-intensive, especially if it fails frequently. Consider optimizing this block by reducing unnecessary computations.

#### Optimization Opportunities
1. **Reduce Redundant Operations**: Move `logger.info` statements outside the `with VigilExperiment` context to avoid repeated logging.
2. **Optimize Exception Handling**: Use a try-except-finally block to ensure resources are cleaned up properly, even if an exception occurs.

```python
try:
    ensemble = validate_ensemble(self._output_dir, split.X_test, split.y_test)
    experiment.log_metrics({
        "ensemble_precision": ensemble.precision,
        "ensemble_recall": ensemble.recall,
        "ensemble_f1": ensemble.f1,
        "ensemble_pr_auc": ensemble.pr_auc,
        "ensemble_roc_auc": ensemble.roc_auc,
    })
    passed = ensemble.passed_gates
except Exception as exc:
    logger.exception("Ensemble validation failed")
    print(
        f"  WARNING: validation raised {type(exc).__name__}: {exc}",
        file=sys.stderr,
    )
finally:
    for name in (AE_FILENAME, RF_FILENAME, IF_FILENAME, SCALER_FILENAME, THRESHOLD_FILENAME):
        experiment.log_artifact(self._output_dir / name)
```

#### Resource Usage Concerns
- Ensure that all files and connections are properly closed using context managers.
- Use `with` statements to manage file handles and database connections.

By addressing these points, you can improve the efficiency and resource management of your training pipeline.

---

*Generated by CodeWorm on 2026-02-28 12:08*
