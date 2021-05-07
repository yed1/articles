## See what Apache is doing
`pidof httpd | sed 's/ / -p /g' | xargs strace -fp`
