Better test workflow for Cython modules 
=========================================

This is simply a nicer way to include test coverage data in Python modules built with Cython. You can look through this
excellent link http://blog.behnel.de/posts/coverage-analysis-for-cython-modules.html to find how to the baisc workflow works. One flaw in thi approach is that we don't want to embed these directives directly in our Cython source files, since the performance impact would be severe in some cases, and we would have to edit our files depending on whether we're doing a test or production build of the module. Here's a better way to do it.

First, you can get rid of the the `linetrace` directive by passing it to the `cythonize` function in your `setyp.py`:
```python
 ext_modules = cythonize(ext_modules, compiler_directives={'linetrace': True})
```

Next, we need find a way to pass the `CYTHON_TRACE_NOGIL=1` macro to `distutil`. This is where the `build_ext` command comes into place. It has an option called `--define` that you can pass to define C preprocessor macros. So, to build your Cython files for line-tracing with, just execute 
```
python setup.py build_ext --inplace --define CYTHON_TRACE
``` 
To run coverage tests, you can just create a small shell script that runs the line above. It will simply force the building of the extension and launch the tests, using the `test` directory. I'm using `pytest` in this example:
```bash
#!/bin/bash
python setup.py build_ext --force --inplace --define CYTHON_TRACE
opts+=(tests/)
opts+=(-v
       --cov=name
       --cov-report term-missing
       --log-format="%(asctime)s %(levelname)s %(thread)d %(message)s")

pytest "${opts[@]}"
```

Now you can keep your Cython sources directive-free and . Note that running `python setup.py install` (or `python setup.py build_ext --inplace`) will simply build the production version of your module, just as it did before.  
