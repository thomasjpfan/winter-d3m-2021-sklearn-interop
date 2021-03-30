# Winter D3M 2021 - sklearn interop

[Link to slides](https://thomasjpfan.github.io/winter-d3m-2021-sklearn-interop/)

## Metadata required for D3M

1. Data type of an estimators' parameters.
    - This was manually curated in [sk_typing](https://github.com/thomasjpfan/sk-typing) using Python typing.
    - This was difficult to get into core scikit-learn, because there is not a convincing use case for Python typing. [#17799](https://github.com/scikit-learn/scikit-learn/pull/17799)
2. Data type of an estimators' attributes.
    - This includes the public and **private** attributes.
    - Public attributes can be documented, but private attributes would need to be
      discovered automatically through training the estimator. This is done in [d3m_estimator_to_primitive](https://github.com/thomasjpfan/d3m_estimator_to_primitive).
4. Semantic types for parameters
    - Tuning
    - Control
    - Resource
5. English description of hyper-parameters
    - Can be automatically parsed as shown in [d3m_estimator_to_primitive](https://github.com/thomasjpfan/d3m_estimator_to_primitive).
6. Hyper-parameters to tune (Optional)
7. Search spaces (Optional)
    - Can be defined in a yaml as shown in [d3m_estimator_to_primitive](https://github.com/thomasjpfan/d3m_estimator_to_primitive).
    - This was manually curated in [sksearchspace](https://github.com/thomasjpfan/sksearchspace/tree/0.22.2.post1)
    - It was shown that we can automatically perform hyperparameter searches by using  curated search spaces using the scikit-learn API. Using a `AutoHalvingRandomSearchCV` from [sksearchspace](https://github.com/thomasjpfan/sksearchspace/tree/AutoHalvingRandomSearchCV_0.24.dev0).
8. Obtain the input data types of a estimator. This can be done with scikit-learn, using estimator tags. For encoders, they expect categorical data:

```python
from sklearn.preprocessing import OneHotEncoder
ohe =OneHotEncoder()
ohe._get_tags()['X_types']
# ['categorical']
```

For vectorizers that expect string input:

```python
from sklearn.feature_extraction.text import CountVectorizer
vect = CountVectorizer()
vect._get_tags()['X_types']
# ['string']
```

## License

This repo is under the [MIT License](LICENSE).
