# yield

```
def fab(max):
    n, a, b, ret = 0, 0, 1,[0]
    while n < max:
        print(b)
        ret.append(b)
        a, b = b, a + b
        n += 1
    return ret

print(fab(10))
```

```
class Fab(object):
    def __init__(self, max):
        self.max = max
        self.n, self.a, self.b = 0, 0, 1

    def __iter__(self):
        return self

    def next(self):
        if self.n < self.max:
            r = self.b
            self.a, self.b = self.b, self.a + self.b
            self.n = self.n + 1
            return r
        raise StopIteration()

for i in Fab(5):
    print(i)
```

```
def fab(max):
    n, a, b = 0, 0, 1
    while n < max:
        yield b
        a, b = b, a + b
        n += 1

print([i for i in fab(5)])
print(type(fab(5)))
print(fab(5).next())

for i in fab(5):
    print(i)
```

```
def yani():
    i = 1
    print('aaa')
    return i
    print('bbb')

ret = yani()
print(ret)
```

```
def fab(max):
    n, a, b = 0, 0, 1
    L = []
    while n < max:
        L.append(b)
        a, b = b, a+b
        n += 1
    return L

print(fab(10))
for i in fab(10):
    print(i)

for i in [0,1,2,3,4,5,6,7,8,9]:
    print(i)

for i in xrange(10):
    print(i)
```

```
class Fab(object):
    def __init__(self, max):
        self.max = max
        self.n, self.a, self.b = 0, 0, 1

    def __iter__(self):
        return self

    def next(self):
        if self.n < self.max:
            r = self.b
            self.a, self.b = self.b, self.a + self.b
            self.n = self.n + 1
            return r
        raise StopIteration()

print(Fab(10))
for i in Fab(10):
    print(i)
```

```
def fab(max):
    n, a, b = 0, 0, 1
    while n < max:
        print('aaa')
        yield b
        print('bbb')
        a, b = b, a+b
        n += 1

print(fab(10))

for i in fab(10):
    print(i)

print(dir(fab(10)))

f = fab(3)
print(f.next())
print("\n")
print(f.next())
print("\n")
print(f.next())
print("\n")
print(f.next())
```

