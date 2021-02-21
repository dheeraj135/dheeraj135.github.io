---
title: "Demystifying Java Reflective Calls"
categories: 
    - MTP
tags:
    - MTP
    - Soot
    - Reflective Calls
    - Java
---

## What the heck are Reflective Calls?
In simple terms, Reflective calls allows the programmer to access Class constructors and methods using their name as a string. For example, if you want to create an object class ```C1```, you can do:

```java
class C1{
  String s;
  public C1(String t) {
    this.s = t;
  }

  private void tmethod(int a, int b,int c) {
    System.out.println(a+b+c);
  }
}
class Test{
  public static void main(String[] args) {
    Class cls = Class.forName("C1");
    Constructor con = cls.getConstructor(String.class);
    C1 obj = (C1) con.getInstance("test-string");// ST1
```

Now, ideally we cannot call the tmethod function because it is private. Using reflection we can call this method.

```java
Method meth = cls.getDeclaredMethod("tmethod", int.class, int.class, int.class);
meth.setAccessible(true);
meth.invoke(obj, 4, 5, 6);// ST2
```

We can invoke this function in one more way.

```java
meth.invoke(obj, new Object[]{4, 5, 6}); // ST3
```

## Why are Reflective calls troublesome?

As you can see from the object snippets, I could easily get the class name at runtime and change the fields at runtime. This makes life of static analyser quite difficult.

## Using tamiflex and Soot

First to analyse such codes, we need tamiflex. Download/build the tamiflex's PlayOutAgent(poa), and use it as:

```bash
javac *.java #Compile all the java files.
java -javaagent:poa-trunk.jar -cp . Refl # Here, Refl is the name of main class, change it accordingly.
or
java -javaagent:poa-trunk.jar -jar jar-name.jar Refl # use -help to get full information.
```

This dumps the ```refl.log``` and class files in ```out/``` directory.

Now, we can analyse it in soot. Look at Soot documentation's article on analysing dacapo benchamrk for more information.

## Jimple and Absurd Call Graph

From the above code snippet you can infer that, the number of arguments in caller statement and callee statement are not going to match. In Soot, all reflective calls are represented as ```REFL_METHOD_INVOKE``` or ```REFL_CONSTRUCTOR_NEWINSTANCE```.

In Jimple, all statements of type ST2 are converted to ST3, i.e. the varargs are converted to an array. The ```getInstance``` also supports varargs. All statements of type ST1 are converted to:

```java
C1 obj = (C1) con.getInstance(new Object[] {"test-instance"});
```

### Handling REFL_CONSTRUCTOR_NEWINSTANCE

In jimple all edges of this type looks like:
```
getInstance(Object[] args) -> C1(arg1, arg2, arg3, ... )
```

Now, to find which all objects can be passed to arg1 of C1 using reflection, we can do :

```java
Value GetObjects(Unit u, int parameterNumber, SootMethod src) {
  InvokeExpr expr;
  if ( u instanceof JInvokeStmt) {
    expr = ((JInvokeStmt)u).getInvokeExpr();
  }
  else if ( u instanceof JAssignStmt) {
    expr = (InvokeExpr)(((JAssignStmt)u).getRightOp());
  }
  Value arg;
  arg = expr.getArg(parameterNumber);
}

List<Value> getObjectsPassed(SootMethod meth, int parameterNumber) { 
  Iterator<Edge> iter = cg.edgesInto(meth);   // meth is SootMethod 

  List<Value> ans = new List<>();
  while(iter.hasNext()) {
    Edge edge = iter.next();

    if (edge.kind() == Kind.REFL_CONSTR_NEWINSTANCE) {
      Value val = GetObjects(edge.srcUnit(), 0, edge.src());
      // Get the value in val's parameterNumber-th index. How to get this?
      // push to ans
    }
  }
  return ans;
}
```

### Handling REFL_METHOD_INVOKE

In jimple all edges of this type looks like:
```
invoke(obj, Object[] args) -> C1(arg1, arg2, arg3, ... )
```

Similiar to REFL_CONSTRUCTOR_NEWINSTANCE, we do the exact same thing with a minor change:

```java
Value val = GetObjects(edge.srcUnit(), 1, edge.src());
```

Argument 0 is actually the parameter -1, i.e. the object itself. Argument 1 contains all the parameters.

