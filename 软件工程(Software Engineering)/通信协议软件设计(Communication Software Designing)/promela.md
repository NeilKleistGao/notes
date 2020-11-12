### Processes
使用`init`创建一个初始化进程：
```
init {
	printf("passed first test!\n")
}
```

使用`proctype`定义一个进程执行的内容，并使用`run`执行：
```
proctype A() {
	printf("ruA")
}

init {
	printf("passed first test!\n")
	run A();
}
```

可以使用`active`使其自动创建对应的进程并执行：
```
active proctype A() {
	printf("ruA")
}
```

### Variables
可以像这样定义基本的变量和类型：
```
bit a; // 0 or 1
bool flag; // 0 or 1
byte count; // from 0 to 255
short s; // 16bits
int msg; // 32bits
byte aa[114]; // arr

typedef Foo {
	short a1;
	int a2;
}
```

### Statements
`skip`相当于Python中的`pass`：
```
active proctype A() {
	skip;
	printf("ruA")
}
```

可以单独写一句逻辑表达式，当表达式为真时才会执行下去，否则阻塞：
```
active proctype A() {
	skip;
	1 == 2;
	printf("ruA") // 永远不会执行
}
```

也可以使用`assert`进行断言：
```
active proctype A() {
	skip;
	assert(1 == 2); // 触发断言
	printf("ruA")
}
```

`if`语句的条件使用`::`给出，使用`->`给出执行语句：
```
active proctype A() {
	byte a = 2;
	if
	:: a == 2 -> printf("wryy")
	:: a == 3 -> printf("wryyy")
    :: else -> skip
	fi;
}
```

`do`语句可以进行循环，使用`::`判断条件，`break`跳出：
```
active proctype A() {
	byte i = 0;
	do
	:: (i < 10) -> i++; printf("%d", i);
	:: (i >= 10) -> break;
	od;
}
```

### channel
声明一个容量为1的，内容为`byte`的信道：
```
chan data = [1] of {byte};
```

使用`?`从信道中取出内容，使用`!`发送，以此进行信息交换：
```
chan data = [1] of {byte};

active proctype A() {
	byte a = 5;
	data!a;
}

active proctype B() {
		byte b;
		data?b;
		printf("%d", b);
	}
```

如果将信道容量改为0，可以起到进程同步的作用。

### others
+ `timeout`负责处理超时事件
+ `goto`同C语言`goto`
+ `#define`同C语言