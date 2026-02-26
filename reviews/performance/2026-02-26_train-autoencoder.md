# train_autoencoder

**Type:** Performance Analysis
**Repository:** Cybersecurity-Projects
**File:** PROJECTS/advanced/ai-threat-detection/backend/ml/train_autoencoder.py
**Language:** python
**Lines:** 16-111
**Complexity:** 7.0

---

## Source Code

```python
def train_autoencoder(
    X_normal: np.ndarray,
    epochs: int = 100,
    batch_size: int = 256,
    lr: float = 1e-3,
    percentile: float = 99.5,
    val_split: float = 0.15,
    patience: int = 10,
) -> dict[str, Any]:
    """
    Train the autoencoder on normal-only traffic and calibrate the
    anomaly detection threshold.
    """
    input_dim = X_normal.shape[1]

    split_idx = int(len(X_normal) * (1 - val_split))
    X_train_raw = X_normal[:split_idx]
    X_val_raw = X_normal[split_idx:]

    scaler = FeatureScaler()
    X_train_scaled = scaler.fit_transform(X_train_raw)
    X_val_scaled = scaler.transform(X_val_raw)

    train_tensor = torch.from_numpy(X_train_scaled)
    val_tensor = torch.from_numpy(X_val_scaled)

    train_loader = DataLoader(
        TensorDataset(train_tensor),
        batch_size=batch_size,
        shuffle=True,
        drop_last=len(train_tensor) > batch_size,
    )

    model = ThreatAutoencoder(input_dim=input_dim)
    optimizer = torch.optim.AdamW(
        model.parameters(), lr=lr, weight_decay=1e-5, betas=(0.9, 0.999)
    )
    scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
        optimizer, mode="min", factor=0.5, patience=5, min_lr=1e-6
    )

    history: dict[str, list[float]] = {"train_loss": [], "val_loss": []}
    best_val_loss = float("inf")
    best_state = None
    epochs_without_improvement = 0

    for epoch in range(epochs):
        model.train()
        epoch_loss = 0.0
        n_batches = 0

        for (batch,) in train_loader:
            reconstructed = model(batch)
            loss = torch.nn.functional.mse_loss(reconstructed, batch)
            optimizer.zero_grad()
            loss.backward()
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
            optimizer.step()
            epoch_loss += loss.item()
            n_batches += 1

        avg_train_loss = epoch_loss / max(n_batches, 1)
        history["train_loss"].append(avg_train_loss)

        model.eval()
        with torch.no_grad():
            val_reconstructed = model(val_tensor)
            val_loss = torch.nn.functional.mse_loss(val_reconstructed, val_tensor).item()
        history["val_loss"].append(val_loss)

        scheduler.step(val_loss)

        if val_loss < best_val_loss:
            best_val_loss = val_loss
            best_state = {k: v.clone() for k, v in model.state_dict().items()}
            epochs_without_improvement = 0
        else:
            epochs_without_improvement += 1

        if epochs_without_improvement >= patience:
            break

    if best_state is not None:
        model.load_state_dict(best_state)

    model.eval()
    with torch.no_grad():
        val_errors = model.compute_reconstruction_error(val_tensor)
    threshold = float(np.percentile(val_errors.numpy(), percentile))

    return {
        "model": model,
        "scaler": scaler,
        "threshold": threshold,
        "history": history,
    }
```

---

## Performance Analysis

### Performance Analysis

**Time Complexity:** The primary bottleneck is the training loop, which iterates over `epochs` (default 100). Each epoch involves multiple batches, leading to a time complexity of \(O(E \times B)\), where \(E\) is the number of epochs and \(B\) is the batch size.

**Space Complexity:** The space complexity is primarily determined by the storage of training and validation data tensors. Assuming each tensor has dimensions `N x D`, the total space used for these tensors is \(O(N \times D + B)\), where \(N\) is the number of samples, and \(D\) is the input dimension.

**Bottlenecks or Inefficiencies:**
1. **Redundant Operations:** The `torch.nn.utils.clip_grad_norm_` call in each training batch is redundant since it's applied to all parameters.
2. **Resource Usage Concerns:** The memory usage can be high, especially if the dataset is large and multiple tensors are stored simultaneously.

**Optimization Opportunities:**
1. **Remove Redundant Operations:** Clip gradients only once per epoch instead of per batch.
2. **Efficient Memory Management:** Use `torch.utils.data.DataLoader` with `pin_memory=True` for faster data transfer to GPU, if applicable.
3. **Caching:** Cache the scaler and model state dictionaries to avoid repeated computations.

**Concrete Optimizations:**
1. Move `torch.nn.utils.clip_grad_norm_` outside the batch loop:
   ```python
   optimizer.zero_grad()
   loss.backward()
   torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
   for (batch,) in train_loader:
       reconstructed = model(batch)
       # ... rest of the training code ...
   ```

2. Ensure efficient memory management by using `pin_memory=True` when creating data loaders.

By addressing these points, you can improve both time and space efficiency, making your training process more robust and performant.

---

*Generated by CodeWorm on 2026-02-26 01:12*
