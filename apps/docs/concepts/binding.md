# Binding

**Binding** is the registration of your services into your dependency injection (DI)  container.

## Auto-binding

The easiest way to register your class for automatic dependency injection, is by
applying the `@injectable()` decorator to your class:

```ts twoslash
import { injectable } from "@needle-di/core";

@injectable()
class FooService {
  // ...
}
```

This will automatically bind `FooService` as a singleton service. To request it from your service,
you can use the `.get()` method on the [container](./containers):

```ts twoslash
import { Container } from "@needle-di/core";
import { FooService } from "./foo.service";

const container = new Container();
const fooService = container.get(FooService);
//     ^?
```

* Its construction is **lazy**: it will only be created when you request it from the container.
* It is also a **singleton**: the first time a `FooService` is injected, a new instance is constructed, but it will
  reuse this instance whenever it needs to be injected again.

> [!NOTE]
> Since Needle DI uses native [ECMAScript decorators](https://github.com/tc39/proposal-decorators)
> (which are currently in [stage 3](https://github.com/tc39/proposals#stage-3)), you will need to transpile your code in
> order to use it in a browser or in Node.js.
>
> All modern transpilers (including [TypeScript], [esbuild], [Webpack], [Babel]) do have support for stage 3 decorators.
> If you
> don't want to depend on transpilation, you can bind your services [manually](#manual-binding) instead, without using
> decorators.

[TypeScript]: https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/#decorators

[esbuild]: https://github.com/evanw/esbuild/releases/v0.21.0

[Webpack]: https://stackoverflow.com/a/37616418/1116452

[Babel]: https://stackoverflow.com/a/37616418/1116452

## Manual binding

If you don't want to use the `@injectable()` decorator, for example if you don't want to use decorators or you want to
bind a class that you cannot decorate (from another library), you can manually register your service with the `.bind()`
method:

```ts twoslash
import { Container } from "@needle-di/core";
import { FooService } from "./foo.service";

const container = new Container();

container.bind(FooService);

const fooService = container.get(FooService);
//    ^?
```

This is the same as applying a decorator to `FooService`.

## Clear binding

To clear a binding, you can use the `.unbind()` or `.unbindAll()` method. This will also remove any instances of the
service from the container.

## Wildcard binding

Sometimes it is useful to generate _providers_ dynamically on the fly. For example, it can be used with universal mocks like [`ts-mockito`](https://github.com/johanblumenberg/ts-mockito).

```ts
import { imock } from "@johanblumenberg/ts-mockito";
import { FooService } from "./foo.service";
import { FooRepository } from "./foo.repository";

const container = new Container();
const mocks = new WeakMap();

container.wildcardBind({
  providerFactory: (token) => {
    const mock = imock(MockPropertyPolicy.Throw);
    mocks.set(token, mock);
    return [{ provide: token, useValue: instance(mock) }];
  },
});

const instance = container.get(FooService);

const repository = mocks.get(FooRepository)!;

when(repository.findOne("testid")).thenReturn({ data: "sample value" });
```

Wildcard binds are tested after regular binds in the order they were registered.
It is possible to ignore token by returing `undefined` from the providerFactory.

---

There are many different ways to bind services,
check out the section about [providers](./providers) to learn more. 
