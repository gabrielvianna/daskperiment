Backend
=======

`daskperiment` uses `Backend` classes to define how and where experiment results are saved. Currently, following backends are supported.

* `LocalBackend`: Information is stored in local files. This is for personal
  usage with single PC. This backend doesn't intend to share information with
  others and move file(s) to another PC.
* `RedisBackend`: Information is stored in Redis. If you setup redis-server,
  information can be shared in a small team and between some PCs.

You can specify required `Backend` via `backend` keyword in `Experiment` instanciation.

LocalBackend
------------

`LocalBackend` saves information as local files. When you create `Experiment` instance without `backend` argument, the `Experiment` uses `LocalBackend` to save its information.

.. note::

   Following examples omit unrelated logs.

.. code-block:: python

   >>> import daskperiment
   >>> daskperiment.Experiment('local_default_backend')
   ... [INFO] Creating new cache directory: /Users/sinhrks/Git/daskperiment/daskperiment_cache/local_default_backend
   ... [INFO] Initialized new experiment: Experiment(id: local_default_backend, trial_id: 0, backend: LocalBackend('daskperiment_cache/local_default_backend'))
   ...
   Experiment(id: local_default_backend, trial_id: 0, backend: LocalBackend('daskperiment_cache/local_default_backend'))

To change the directory location, passing `pathlib.Path` instance as `backend` makes `LocalBackend` with custom location.

.. code-block:: python

   >>> import pathlib
   >>> daskperiment.Experiment('local_custom_backend', backend=pathlib.Path('my_dir'))
   ... [INFO] Creating new cache directory: /Users/sinhrks/Git/daskperiment/my_dir
   ... [INFO] Initialized new experiment: Experiment(id: local_custom_backend, trial_id: 0, backend: LocalBackend('my_dir'))
   ...
   Experiment(id: local_custom_backend, trial_id: 0, backend: LocalBackend('my_dir'))


The following table shows information and its saved location under cache directory specified in `LocalBackend`.

============================================= ========== ===================
Information                                   Format     Path
============================================= ========== ===================
Experiment status (internal state)            Pickle     <experiment id>.pkl
Experiment history                            Pickle     <experiment id>.pkl
Persisted results                             Pickle     persist/<experiment id>_<function name>_<trial id>.pkl
Metrics                                       Pickle     <experiment id>.pkl
Function input & output hash                  Pickle     <experiment id>.pkl
Code contexts                                 Text       code/<experiment id>_<trial id>.py
Platform information                          Text(JSON) environmemt/device_<experiment id>_<trial id>.json
CPU information                               Text(JSON) environmemt/cpu_<experiment id>_<trial id>.json
Python information                            Text(JSON) environmemt/python_<experiment id>_<trial id>.json
NumPy information (`numpy.show_config`)       Text       environmemt/numpy_<experiment id>_<trial id>.txt
SciPy information (`scipy.show_config`)       Text       environmemt/scipy_<experiment id>_<trial id>.txt
pandas information (`pandas.show_versions`)   Text       environmemt/pandas_<experiment id>_<trial id>.txt
conda information (`conda info`)              Text       environmemt/conda_<experiment id>_<trial id>.txt
Git information                               Text(JSON) environmemt/git_<experiment id>_<trial id>.json
Python package information                    Text       environmemt/requirements_<experiment id>_<trial id>.txt
============================================= ========== ===================


RedisBackend
------------

`RedisBackend` saves information using `Redis`. To use `RedisBackend`, simple way is specifying Redis URI as `backend` argument.

.. code-block:: python

   >>> daskperiment.Experiment('redis_uri_backend', backend='redis://localhost:6379/0')
   ... [INFO] Initialized new experiment: Experiment(id: redis_uri_backend, trial_id: 0, backend: RedisBackend('redis://localhost:6379/0'))
   ...
   Experiment(id: redis_uri_backend, trial_id: 0, backend: RedisBackend('redis://localhost:6379/0'))

Or, you can use `redis.ConnectionPool`.

   >>> import redis
   >>> pool = redis.ConnectionPool.from_uri('redis://localhost:6379/0')
   >>> daskperiment.Experiment('redis_pool_backend', backend=pool)
   ... [INFO] Initialized new experiment: Experiment(id: redis_pool_backend, trial_id: 0, backend: RedisBackend('redis://localhost:6379/0'))
   ...
   Experiment(id: redis_pool_backend, trial_id: 0, backend: RedisBackend('redis://localhost:6379/0'))


The following table shows information and its saved location under Redis database specified in `RedisBackend`.

============================================= ========== ===================
Information                                   Format     Key
============================================= ========== ===================
Experiment status (internal state)            Text       <experiment id>:trial_id
Experiment history (parameters)               Pickle     <experiment id>:parameter:<trial id>
Experiment history (results)                  Pickle     <experiment id>:history:<trial id>
Persisted results                             Pickle     <experiment id>:persist:<function name>:<trial id>
Metrics                                       Pickle     <experiment id>:metric:<metric name>:<trial id>
Function input & output hash                  Text       <experiment id>:step_hash:<function name>-<input hash>
Code contexts                                 Text       <experiment id>:code:<trial id>
Platform information                          Text(JSON) <experiment id>:device:<trial id>
CPU information                               Text(JSON) <experiment id>:cpu:<trial id>
Python information                            Text(JSON) <experiment id>:python:<trial id>
NumPy information (`numpy.show_config`)       Text       <experiment id>:numpy:<trial id>
SciPy information (`scipy.show_config`)       Text       <experiment id>:scipy:<trial id>
pandas information (`pandas.show_versions`)   Text       <experiment id>:pandas:<trial id>
conda information (`conda info`)              Text       <experiment id>:conda:<trial id>
Git information                               Text(JSON) <experiment id>:git:<trial id>
Python package information                    Text       <experiment id>:requirements:<trial id>
============================================= ========== ===================
