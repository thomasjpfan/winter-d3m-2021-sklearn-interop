+++
title = "Scikit-learn D3M Interop"
outputs = ["Reveal"]
+++

# Scikit-learn D3M Interop

{{% grid middle %}}

{{< g 1 >}}
{{< figure src="images/d3m2.png" height="240px" >}}
{{< /g >}}
{{< g 1 >}}
{{< figure src="images/scikit-learn-logo.png" height="240px" >}}
{{< /g >}}
{{< g 1 >}}
{{< figure src="images/columbia.jpeg" height="240px" >}}
{{< /g >}}
{{% /grid %}}

## Winter 2021
### **Thomas J. Fan @ Columbia University**

---

# Main Goal

## Auto-generate a D3M Primitive from **ANY** Scikit-learn compatible estimator

---

# High Level Overview

1. Library author already has a scikit-learn compatible estimator
2. {{< frag c="Contributor using D3M's framework wants to add primitive" >}}
3. {{< frag c="Contributor writes the types for hyper-parameter and attributes" >}}
4. {{< frag c="Contributor adds additional metadata" >}}
5. {{< frag c="Run auto-generation script developed by D3M to generate primitive" >}}

---

# Goal
**Provide a CLI tool**

```bash
pip install d3m_estimator_to_primitive

d3m_estimator_to_primitive --meta additional_metadata.yaml xgboost.XGBClassifier
```

would generate a `XGBClassifierPrimitive.py`

### **Allow ANY Scikit-learn compatible estimator be a primitive**

---

# What has been developed?

- We already have some of this work in **sklearn-wrap**
- How do we leverage this code?

---

## 1. ✅ Library author already has a scikit-learn compatible estimator
## 2. ✅ Contributor using D3M's framework wants to add primitive

---

{{% section %}}

# 3. Contributor writes the types for hyper-parameter and attributes

---

# Metadata using Python Typing

### [github.com/thomasjpfan/sk-typing](https://github.com/thomasjpfan/sk-typing)

```python
class KMeans:

    cluster_centers_: np.ndarray
    labels_: np.ndarray
    inertia_: float
    n_iter_: int

    def __init__(
        self,
        n_clusters: int = 8,
        init: Union[Literal["k-means++", "random"], Callable, ArrayLike] = "k-means++",
        ...
```

---

# Converts to a json format for sklearn_wrap

```json
  "KMeans": {
    "name": "sklearn.cluster._kmeans.KMeans",
    "common_name": "KMeans",
    "description": "K-Means clustering. Read more in the :ref:`User Guide <k_means>`.",
    "sklearn_version": "0.22.2.post1",
    "Hyperparams": [
      {
        "type": "Hyperparameter",
        "name": "n_clusters",
        "init_args": {
          "semantic_types": [
            "n_clusters"
          ],
          "_structural_type": "int",
          "default": 8,
          "description": "The number of clusters to form as well as the number of centroids to generate."
        }
      },
```

{{% /section %}}

---

{{% section %}}

# 4. Contributor Adds Additional metadata

### **We should keep this at to a minimum**

---

# Additional metadata we need now:

- Specify hyperparams_to_tune
- Specify search spaces (Optional)
- **Private attributes**

---

# Specify `hyperparameters_to_tune`

- Unless library authors specify this, the D3M contributor would have to
- Tuning and Control Parameters
- This can be another barrier

---

# Specify search spaces (Optional)

---

## sksearchspace
### [github.com/thomasjpfan/sksearchspace](https://github.com/thomasjpfan/sksearchspace)

> Scikit-learn Search Space Configurations with curated search spaces for scikit-learn estimators.

---

# sksearchspace

sksearchspace uses ConfigSpace for sampling.

```python{1-4|6-7}
from sksearchspace import SearchSpace
from sklearn.tree import DecisionTreeClassifier

estimator_space = SearchSpace.for_sklearn_estimator(DecisionTreeClassifier, seed=42)

estimator_space.sample()
# {'criterion': 'entropy','min_samples_leaf': 15, 'min_samples_split': 11}
```

---

# Search spaces in ML

ML ecosystem has many frameworks for defining spaces

- ray tune - https://docs.ray.io/en/master/tune/index.html
- keras-tuner - https://keras-team.github.io/keras-tuner/
- ConfigSpace - https://automl.github.io/ConfigSpace/master/
- scikit-learn - uses dicts + scipy distributions

### Requires tool to translate into D3M Hyperparameters

---

# Private attributes !

#### Example: `SKKNeighborsClassifier`

```python
class Params(params.Params):

    _fit_method: Optional[str]
    n_samples_fit_: Optional[int]
    _fit_X: Optional[ndarray]
    _tree: Optional[object]
    ...
```

- Attributes starting with `_` are undocumented.
- Brittle: private attributes can change without warning
- Providing this metadata requires one to dive into the source code
- Huge barrier for creating primitives

---

# Why do we have this?

D3M uses `set_params` and `get_params` to serialized and deserialize the primitive

```python
    def set_params(self, *, params: Params) -> None:
        ...

    def get_params(self) -> Params:
        ...
```

---

# First Solution

- Treat the estimator like a semi-transparent box
- Provide access to the public attributes
- This is how it works for `xgboost` and `lightgbm`'s boosters in the `common-primitives`

```python
self._learner.set_params(_Booster=params['booster'], n_classes_=params['n_classes'],
```

- **May break downstream code**

---

# Second Solution

- Figure out the private attributes by fitting the estimator and inspecting the model
- Should work 95% of the time.
- **Will not break down stream code**

{{% /section %}}

---

{{% section %}}

# Work That Has Been Done

**[github.com/thomasjpfan/sk-typing](https://github.com/thomasjpfan/sk-typing)**: Specify typing for all scikit-learn estimators

**[github.com/thomasjpfan/sksearchspace](https://github.com/thomasjpfan/sksearchspace)**: Specify search spaces for all scikit-learn estimators

---

# Ongoing Work

- Try to reuse as much as possible from `sklearn-wrap`
- Specify and document how to write the additional metadata

---

# Goal
**Provide a CLI tool**

```bash
pip install d3m_estimator_to_primitive

d3m_estimator_to_primitive --meta additional_metadata.yaml xgboost.XGBClassifier
```

would generate a `XGBClassifierPrimitive.py`

### **Allow ANY Scikit-learn compatible estimator be a primitive**

{{% /section %}}
