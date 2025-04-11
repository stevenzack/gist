Index

- [dlv](#dlv)
- [gdb](#gdb)
- [assembly](#assembly)

# dlv

```sh
go install github.com/go-delve/delve/cmd/dlv@latest
dlv debug main.go
```

```sh
> b main.main //break main.main() function
> n // next line
> s // step into subfunctions
> stack // print stack
> gr // print goroutines
> args // print arguments
> locals // print local variable's values
```

---

## Dlv Remote debugging

Build the main package
```sh
go build -o app .
```

Put it onto your server
```sh
sftp root@127.0.0.1
>put app 
>put dlv
```

In your server, launch dlv
```sh
dlv --listen=:80 --headless=true --log=true --log-output=debugger,debuglineerr,gdbwire,lldbout,rpc --accept-multiclient --api-version=2 exec ./app &
```

In VSCode, create `.vscode/launch.json`
```json
{
	"version": "0.2.0",
	"configurations": [
		{
			"name": "Attach",
			"type": "go",
			"request": "attach",
			"mode": "remote",
			"remotePath": "",
			"port":80,
			"host":"127.0.0.1",
			"showLog": true,
			"trace": "log",
			"logOutput": "rpc"
		}
	]
}
```

Add a break point in your code, and now you can do remote debugging in VSCode

# gdb

```main.c
#include <stdio.h>
#include <unistd.h>
int add(int a,int b){
    return a+b;
}

int main(){
    sleep(20000);
    int sum;
    for(int i=0;i<10;i++){
        sum+=add(i, i*2);
    }
    printf("%d\n",sum);
}
```
```sh
gcc -g main.c
gdb ./a.out
```

```sh
>l #list
>c #continue
>n #next
>b #breakpoint
```

# Assembly

Print assembly code
```sh
go build -gcflags -S .
```
