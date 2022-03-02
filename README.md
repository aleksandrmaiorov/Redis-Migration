# Redis Migration Script

# How to Run

  - Please define your redis servers.
 ```sh
$ export OLD="redis-cli -h old-server.example.com"
$ export NEW="redis-cli -h new-server.example.com"
```
  - Before we continue we should check that both servers are accessible and responding:
 ```sh
 $OLD PING
# PONG

$NEW PING
# PONG
```
  - Great! Now let's push the big red button. Or, at least look at it:
``` sh 
    vi redis-migration.sh
```

```sh
for KEY in $($OLD --scan); do
    $OLD --raw DUMP "$KEY" | head -c-1 > /tmp/dump
    TTL=$($OLD --raw TTL "$KEY")

    case $TTL in
        -2)
            $NEW DEL "$KEY"
            ;;
        -1)
            $NEW DEL "$KEY"
            cat /tmp/dump | $NEW -x RESTORE "$KEY" 0
            ;;
        *)
            $NEW DEL "$KEY"
            cat /tmp/dump | $NEW -x RESTORE "$KEY" "$TTL"
            ;;
    esac

    echo "$KEY (TTL = $TTL)"
done
```
```sh
$ bash redis-migration.sh
```
  - In order to be able to complete it please use one of the servers(QA01,Stage,DevNext)
  

  - If you see an error similar to "(error) MOVED 660 10.1.174.29:6379" it's because you are running against a cluster and will need to add the "-c" option to your redis-cli command.
