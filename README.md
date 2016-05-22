## Framework for Testing WAFs (FTW)

##### Purpose 
This project was created by researchers from ModSecurity and Fastly to help provide rigorous tests for WAF rules. It uses the OWASP Core Ruleset V3 as a baseline to test rules on a WAF. Each rule from the ruleset is loaded into a YAML file that issues HTTP requests that will trigger these rules. 

Goals / Use cases include:

* Find regressions in WAF deployments by using continuous integration and issuing repeatable attacks to a WAF
* Provide a testing framework for new rules into ModSecurity, if a rule is submitted it MUST have corresponding positive & negative tests
* Evaluate WAFs against a common, agreeable baseline ruleset (OWASP)
* Test and verify custom rules for WAFs that are not part of the core rule set

## Installation
* `git clone git@github.com:fastly/ftw.git`
* `cd ftw`
* `pip install -r requirements.txt`

## Running Tests
* *start your test web server*
* Create YAML files that point to your webserver with a WAF in front of it
* `py.test test/test_modsecurityv2.py --ruledir test/yaml`

## Running Tests while overriding destination address in the yaml files to custom domain
* *start your test web server*
* `py.test test/test_modsecurityv2.py --ruledir=test/yaml --destaddr=domain.com`

## Run integration test, local webserver, may have to use sudo
* `sudo py.test test/integration/test_logcontains.py -s --ruledir=test/integration/`

## HOW TO INTEGRATE LOGS
1. Create a `*.py` file with the necessary imports, shown in `test/integration/test_logcontains.py`
2. All functions with `test*` in the beginning will be ran by `py.test`, so make a function `def test_somewaf`
3. Implement a class that inherits `LogChecker`
  1. Implement the `get_logs()` function. FTW will call this function after it runs the test, and it will set datetimes of `self.start` and `self.end`
  2. Use the information from the datetime variables to retrieve the files from your WAF, whether its a file or an API call
  3. Get the logs, store them in an array of strings and return it from `get_logs()`
4. Write a testing configuration in the `*.yaml` format as seen in `test/integration/LOGCONTAINSFIXTURE.yaml`, the `log_contains` line requires a string that is a regex. FTW will compile the `log_contains` string from each stage in the YAML file into a regex. This regex will then be used alongside the lines of logs passed in from `get_logs()` to look for a match. The `log_contains` string, then, should be a unique rule-id as FTW is greedy and will pass on the first match. False positives are mitigated from the start/end time passed to the `LogChecker` object, but it is best to stay safe and use unique regexes.
5. For each stage ran the `get_logs()` function is called, so be sure to account for API calls if thats how you retrieve your logs. 
