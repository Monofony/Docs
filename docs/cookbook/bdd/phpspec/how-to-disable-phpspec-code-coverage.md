# How to disable phpspec code coverage

```bash
$ cp phpspec.yml.dist phpspec
```

And just comment the content

```yaml
# extensions:
#    LeanPHP\PhpSpec\CodeCoverage\CodeCoverageExtension: ~
```
