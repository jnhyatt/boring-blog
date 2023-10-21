+++
title = "Dev Log #2"
[taxonomies]
tags = [ "bevy" ]
+++

# Patterns and Anti-patterns

I can't find a legitimate use for the visitor pattern. Essentially, the visitor pattern is a way to model and switch on fixed object variants. That's some imprecise language right there, so let's go a little deeper: imagine you have an AST for a cool new programming language you want to design. The syntax supports import directives, function declarations, and more. You're a die-hard Java fanboy so you model each as a class:

```java
interface AstNode {}

public class ImportDirective implements AstNode {
    public String module;
}

public class FunctionDeclaration implements AstNode {
    public String name;
}
```

This is a fixed set of variants for `AstNode`, meaning none of the users of this API are going to be adding variants. Now, you want to make a compiler, a formatter, a linter, etc. so you're going to have to be able to traverse this hierarchy doing different things -- for example, you want your formatter to read in the code, parse it out, then simply write the properly formatted code back to the file. You also want to be able to do codegen on the hierarchy. You briefly consider adding `String formatted()` and `List<Instruction> codegen()` to `AstNode`, but reject the idea because you are a self-respecting Java developer that knows design patterns! Instead you implement the visitor pattern:

```java
interface AstNode {
    void accept(AstVisitor visitor);
}

interface AstVisitor {
    void visitImportDirective(ImportDirective x);
    void visitFunctionDeclaration(FunctionDeclaration x);
}

class ImportDirective implements AstNode {
    public String module;

    public void accept(AstVisitor visitor) {
        visitor.visitImportDirective(this);
    }
}

class FunctionDeclaration implements AstNode {
    public String name;

    public void accept(AstVisitor visitor) {
        visitor.visitFunctionDeclaration(this);
    }
}
```

With this structure in place, we can now create implementors of `AstVisitor` to handle all the work you wanted to do on your AST:

```java
class AstWriter {
    Writer writer;

    AstWriter(Writer writer) {
        this.writer = writer;
    }

    public void visitImportDirective(ImportDirective x) {
        writer.append("import " + x.module)
    }

    public void visitFunctionDeclaration(FunctionDeclaration x) {
        writer.append("function " + x.name + "() {}");
    }
}

public class Codegen {
    Module module;

    public AstWriter(Writer writer) {
        this.writer = writer;
    }

    public void visitImportDirective(ImportDirective x) {
        module.addDependency(x.module);
    }

    public void visitFunctionDeclaration(FunctionDeclaration x) {
        module.declareFunction(Type.Void, x.name);
    }
}
```

This is starting to look good, right? Now we can format our AST just by doing `astRoot.accept(new AstWriter(fileWriter));`, or we could codegen by writing `astRoot.accept(new Codegen(myModule));`. We can also implement new ways to traverse this hierarchy without modifying the hierarchy itself at all! It also forces us to handle all cases. Watch what happens when we add a new node:

```java
class InterfaceDeclaration implements AstNode {
    public String name;

    public void accept(AstVisitor visitor) {
        // ??? HALP
    }
}
```

How do we implement `accept` for our new variant? We need a `visitInterfaceDeclaration` for `AstVisitor`. Let's add it:

```java
interface AstVisitor {
    void visitImportDirective(ImportDirective x);
    void visitFunctionDeclaration(FunctionDeclaration x);
    void visitInterfaceDeclaration(InterfaceDeclaration x); // NEW
}
```

Uh-oh, all our `AstVisitor` implementors just broke! But that's a good thing. It means we have to go back and consider how each of them should handle the new variant. The visitor pattern has forced us to handle new cases! That's something dynamic casting never would've gotten us. Our code would've just thrown an exception, but only after we got around to adding a test for the new feature. We all want errors as early as possible.

The visitor pattern has a fairly simple implementation in Java, C++, and any other statically-typed language you use. Yes, there's some boilerplate, then there's the double dynamic dispatch from calling two virtual functions. Oh, and don't forget the fact that users that aren't familiar with the pattern might try to subclass your base interface and then get hung up on how to implement `accept`. But if they haven't read *Design Patterns*, that's *their* fault. Visitor pattern is an excellent way to go about this, right?

# No.

I tend to think of things like this as anti-patterns. I'm not going to contest the point that this is probably the best way to get this done in Java. However, when anti-patterns like this show up in your code, I tend to see it as an indicator that your language may be missing features.

In saying this, I am most certainly beating what's left of this long-dead horse. Modern languages have elegantly addressed this with sum types, which are exactly what we were trying to hack into Java for the last few minutes. A sum type is a type that consist of the *sum* of several variants: it can be one of several things. Discriminated unions in F# are a great example of this:

```fsharp
type AstNode = Import | FnDecl | ClassDecl
```

In Kotlin, they're `sealed class`es:

```kt
sealed class AstNode
class Import(val module: String) : AstNode()
class FnDecl(val name: String) : AstNode()
class ClassDecl(val name: String) : AstNode()
```

Rust `enum`s:

```rs
pub enum AstNode {
    Import { module: String },
    FnDecl { name: String },
}
```

Most modern languages have some form of tagged union (Go notably does not have them and I'm not sure why not). 
