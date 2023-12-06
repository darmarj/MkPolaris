# NetCat

## Netcat (or nc) is a command-line utility that reads and writes data across network connections, using the TCP or UDP protocols. It is one of the most powerful tools in the network and system administrators arsenal, and it as considered as a Swiss army knife of networking tools.

### Syntax
```shell
nc [option] host port
```

By default, Netcat will attempt to start a TCP connection to the specified host and port. If you would like toestablish a UDP connection, use the -u option:
```shell
nc -u host port
```

### Example
To scan for open ports in the range 20-80 you would use the following command:
```shell
nc -zv $IP $PORT

# Print only the lines with the open ports, filter the results with the grep command.
nc -zv $IP $PORT 2>&1 | grep succeeded

# Output
Connection to 10.10.8.8 22 port [tcp/ssh] succeeded!
Connection to 10.10.8.8 80 port [tcp/http] succeeded!
```

###
