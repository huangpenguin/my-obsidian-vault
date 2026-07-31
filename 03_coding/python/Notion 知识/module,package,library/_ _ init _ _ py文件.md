---
title: "_ _ init _ _.py文件"
publish: false
tags: ["Python"]
---
# _ _ init _ _.py文件

`__init__.py` 是一个特殊的 Python 文件，用于标识一个目录是一个 Python 包。它的主要作用是初始化包，并可以包含包的初始化代码。以下是对 `__init__.py` 文件的详细解释及其用法示例。

### `__init__.py` 的作用

1. **标识包**：
    - `__init__.py` 文件用于标识一个目录是一个 Python 包。这意味着，如果一个目录包含 `__init__.py` 文件，该目录及其子目录中的模块可以被导入。
2. **包初始化**：
    - `__init__.py` 文件可以包含包的初始化代码，例如初始化包的变量、导入包中的模块等。
3. **控制包的导入行为**：
    - 通过在 `__init__.py` 文件中定义 `__all__` 变量，可以控制 `from package import *` 语句的行为。

### 示例

假设我们有以下目录结构：

```markdown

my_package/
    __init__.py
    module1.py
    module2.py

```

### 示例 1：简单的 `__init__.py` 文件

最简单的 `__init__.py` 文件可以是一个空文件：

```python

# my_package/__init__.py

```

这使得 `my_package` 成为一个可以导入的包：

```python
# script.py
import my_package.module1
import my_package.module2

```

### 示例 2：在 `__init__.py` 文件中初始化包

你可以在 `__init__.py` 文件中初始化包的内容，例如导入包中的模块：

```python
# my_package/__init__.py

from .module1 import some_function
from .module2 import another_function

```

这样，你可以直接从包中导入这些函数：

```python
# script.py
from my_package import some_function, another_function

```

### 示例 3：使用 `__all__` 控制导入行为

通过定义 `__all__` 变量，你可以控制使用 `from package import *` 时导入的内容：

```python
# my_package/__init__.py

__all__ = ['module1', 'module2']

```

这样，当你使用 `from my_package import *` 时，只会导入 `module1` 和 `module2`：

```python

# script.py
from my_package import *

# Now you can use module1 and module2
module1.some_function()
module2.another_function()

```

### 示例：结合前述代码中的 `__init__.py`

结合你之前的代码示例，我们可以创建一个 `__init__.py` 文件来组织和初始化包中的线性模型：

假设我们的目录结构如下：

```markdown

linear_models/
    __init__.py
    _base.py
    _bayes.py
    _coordinate_descent.py
    _glm.py
    _huber.py
    _least_angle.py
    _logistic.py
    _omp.py
    _passive_aggressive.py
    _perceptron.py
    _quantile.py
    _ransac.py
    _ridge.py
    _stochastic_gradient.py
    _theil_sen.py

```

`__init__.py` 文件的内容可以如下：

```python

# linear_models/__init__.py

from ._base import LinearRegression
from ._bayes import ARDRegression, BayesianRidge
from ._coordinate_descent import (
    ElasticNet,
    ElasticNetCV,
    Lasso,
    LassoCV,
    MultiTaskElasticNet,
    MultiTaskElasticNetCV,
    MultiTaskLasso,
    MultiTaskLassoCV,
    enet_path,
    lasso_path,
)
from ._glm import GammaRegressor, PoissonRegressor, TweedieRegressor
from ._huber import HuberRegressor
from ._least_angle import (
    Lars,
    LarsCV,
    LassoLars,
    LassoLarsCV,
    LassoLarsIC,
    lars_path,
    lars_path_gram,
)
from ._logistic import LogisticRegression, LogisticRegressionCV
from ._omp import (
    OrthogonalMatchingPursuit,
    OrthogonalMatchingPursuitCV,
    orthogonal_mp,
    orthogonal_mp_gram,
)
from ._passive_aggressive import PassiveAggressiveClassifier, PassiveAggressiveRegressor
from ._perceptron import Perceptron
from ._quantile import QuantileRegressor
from ._ransac import RANSACRegressor
from ._ridge import Ridge, RidgeClassifier, RidgeClassifierCV, RidgeCV, ridge_regression
from ._stochastic_gradient import SGDClassifier, SGDOneClassSVM, SGDRegressor
from ._theil_sen import TheilSenRegressor

__all__ = [
    "ARDRegression",
    "BayesianRidge",
    "ElasticNet",
    "ElasticNetCV",
    "HuberRegressor",
    "Lars",
    "LarsCV",
    "Lasso",
    "LassoCV",
    "LassoLars",
    "LassoLarsCV",
    "LassoLarsIC",
    "LinearRegression",
    "LogisticRegression",
    "LogisticRegressionCV",
    "MultiTaskElasticNet",
    "MultiTaskElasticNetCV",
    "MultiTaskLasso",
    "MultiTaskLassoCV",
    "OrthogonalMatchingPursuit",
    "OrthogonalMatchingPursuitCV",
    "PassiveAggressiveClassifier",
    "PassiveAggressiveRegressor",
    "Perceptron",
    "QuantileRegressor",
    "Ridge",
    "RidgeCV",
    "RidgeClassifier",
    "RidgeClassifierCV",
    "SGDClassifier",
    "SGDRegressor",
    "SGDOneClassSVM",
    "TheilSenRegressor",
    "enet_path",
    "lars_path",
    "lars_path_gram",
    "lasso_path",
    "orthogonal_mp",
    "orthogonal_mp_gram",
    "ridge_regression",
    "RANSACRegressor",
    "PoissonRegressor",
    "GammaRegressor",
    "TweedieRegressor",
]

```

这样，你就可以方便地导入包中的所有模型：

```python
# script.py
from linear_models import LinearRegression, Ridge, Lasso

# 创建模型实例
linear_model = LinearRegression()
ridge_model = Ridge()
lasso_model = Lasso()
```

### 总结

- `__init__.py` 文件用于标识一个目录是一个 Python 包，并初始化包的内容。
- 它可以包含包的初始化代码和导入包中的模块。
- 可以通过定义 `__all__` 变量来控制 `from package import *` 的导入行为。

通过正确使用 `__init__.py` 文件，可以更好地组织和管理 Python 包，使代码更易读和维护。
