

```python
import sys, os
import logging
import unittest

log = logging.getLogger()

from nose import core, loader

#logging.basicConfig(level=logging.DEBUG)

from types import ModuleType
```


```python
def adder(a, b):
    return a + b

def test_nothing():
    assert 1+2 == 2

def test_adder():
    assert adder(3, 4) == 7
    assert adder(-3, -5) == -8
    
import random
def test_random_generated_tests():
    for _ in range(random.randint(1, 100)):
        yield lambda: None

flake = False
def test_flake():
    global flake
    flake = not flake
    assert flake
```


```python
class MyProgram(core.TestProgram):
    # XXX yuck: copy superclass runTests() so we can instantiate our own runner class;
    # can't do it early because we don't have access to nose's config object.
    def runTests(self):
        self.testRunner = MyRunner(self.config)
        # the rest is mostly duplicate code ;-(
        plug_runner = self.config.plugins.prepareTestRunner(self.testRunner)
        if plug_runner is not None:
            self.testRunner = plug_runner
        self.result = self.testRunner.run(self.test)
        self.success = self.result.wasSuccessful()
        return self.success

class MyResult(unittest.TestResult):
    def make_bar(self, tests, failing):
        return '''<div>
                <div style="background:red; width:%dpx">&nbsp;</div>
                <div style="background:green; width:%dpx">&nbsp;</div>
                </div>''' % (
            failing * 10, (tests - failing) * 10)

    def make_table_of_tests(self, tests):
        table = '<table>'
        for test in tests:
            table += '<tr><td>%s</td><td><pre><![CDATA[%s]]></pre></td>' % (
                str(test[0]), test[1])
        table += '</table>'
        return table
    
    def _repr_html_(self):
        if self.errors or self.failures:
            not_successes = len(self.errors) + len(self.failures)
            return self.make_bar(self.testsRun, not_successes) + \
                '''
                <h2 style="color:red">Errors</h2>%s
                <h2 style="color:red">Failures</h2>%s''' % (
                    self.make_table_of_tests(self.errors),
                    self.make_table_of_tests(self.failures))
        else:
            return self.make_bar(self.testsRun, 0) + '<div>%d/%d&nbsp;tests&nbsp;passed</div>' % (
                self.testsRun, self.testsRun)

class MyRunner(object):
    def __init__(self, config):
        self.config = config

    def run(self, test):
        result = MyResult()
        if hasattr(result, 'startTestRun'):   # python 2.7
            result.startTestRun()
        test(result)
        if hasattr(result, 'stopTestRun'):
            result.stopTestRun()
        self.config.plugins.finalize(result)
        self.result = result
        return result
```


```python
test_module = ModuleType('test_module')
test_module.__dict__.update(get_ipython().user_ns)

ldr = loader.TestLoader()
tests = ldr.loadTestsFromModule(test_module)
#print("discovered: %r" % list(tests._tests))

tprog = MyProgram(argv=['dummy'], exit=False, suite=tests)
tprog.result
```




<div>
                <div style="background:red; width:10px">&nbsp;</div>
                <div style="background:green; width:640px">&nbsp;</div>
                </div>
                <h2 style="color:red">Errors</h2><table></table>
                <h2 style="color:red">Failures</h2><table><tr><td>__main__.test_nothing</td><td><pre><![CDATA[Traceback (most recent call last):
  File "/Users/flatironschool/anaconda/lib/python3.6/site-packages/nose/case.py", line 197, in runTest
    self.test(*self.arg)
  File "<ipython-input-15-ae7cf2e003f7>", line 5, in test_nothing
    assert 1+2 == 2
AssertionError
]]></pre></td></table>




```python

```
