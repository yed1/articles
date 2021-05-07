## break $PATH output into multiple lines, one path per line
* `echo $PATH | python -c "import sys; [sys.stdout.write('\n'.join(line.split(':'))) for line in sys.stdin]"`
* `echo $PATH | tr ':' '\n'`
