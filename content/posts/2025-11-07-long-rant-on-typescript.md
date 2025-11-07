+++
layout = 'post'
title = "A Long Rant on TypeScript"
date = 2025-11-07 02:15:00.000000000+05:45
draft = true
+++

This is going to be a long rant on TypeScript. I've been using typescript for a while now. Let's get on to the good and the bad parts of it. Typescript compiles to JavaScript, and it adds static typing to the language. This is a damage reduction technique to JavaScript's dynamic typing.

## Why? What was the need to add types to TypeScript?
- Better to catch errors at compile time rather than runtime, find something that is broken before it goes to production.
- Improved code readability and maintainability, types serve as documentation for the code.
- Good IDE support, I code with vim and I get a good development experience with typescript.

```typescript
var a = 977;
a = 'Hello'; // Error: Type 'string' is not assignable to type 'number'.
```
Typescript infers the type of variable `a` as number, even if you don't specify. One of the good thing about typescript, if you rename `.js` file to `.ts`, it will infer types for you. Specify errors at compile time if there are any rather than runtime.

Types can be explicitly defined as well.

```typescript
let b: string = 'Hello, TypeScript!';
b = 42; // Error: Type 'number' is not assignable to type 'string'.
```

But, even if there is an error or a warning, the code will still compile to JavaScript, this is one of the controversial part of TypeScript. This is because TypeScript allows you smooth transition from JavaScript on a big project. You can gradually add types to your codebase without breaking anything. But this also means that you can ignore errors and warnings.

As the book "Typescript Deep Dive" says:
> Just giving you a new syntax doesn't help fixing bugs

Things doesn't work out of the box, you need to know more about Typescript to use it effectively.

