
# unison-continuations

continuations for [unison](https://github.com/unisonweb/unison)

## Usage

```
ability Substitution ref where
  jump : ref a -> a ->{Substitution ref} r
  subst : {Substitution ref} (Either (ref a) a)
```

Use `runComp` to evaluate `Substitution` computations.

### `jump`

```
Substitution.jump : ref a -> a ->{Substitution ref} r
```


`jump` replaces the current computation with a previously saved computation using
the supplied computation reference and value.


### `subst`

```
Substitution.subst : {Substitution ref} (Either (ref a) a)
```


`subst` saves the current computation, returning a reference to it. If the computation
is restored, `subst` instead returns the value supplied.


### `callCC`

```
Substitution.callCC : ((a ->{Substitution ref} b) ->{e} a)
                      ->{e, Substitution ref} a
```


`callCC` (call-with-current-continuation) calls a function with the current continuation
as its argument. Calling the named continuation anywhere within its scope will escape
from the computation, even if it is many layers deep within nested computations.


### `setjmp`

```
Substitution.setjmp : a ->{Substitution ref} b ->{Substitution ref} r
```


`setjmp` saves a restoration point for the computation, returning a handle to it.
Using the handle will restore control flow to where `setjmp` was used.


## Examples

```
Substitution.examples.greeter : '{IO} ()
Substitution.examples.greeter =
  'let
    null = cases
      "" -> true
      _  -> false
    when b x = if b then !x else ()
    readNat _ = Nat.fromText !readLine
    p return =
      use Text ++
      printLine "Please enter your name."
      name = !readLine
      when
        (null name) 'let
          printLine "You forgot to tell me your name!"
          !return
      printLine ("Welcome " ++ name ++ "!")
      longjmp = !setjmp
      printLine "How old are you?"
      age =
        match !readNat with
          Some x -> x
          None   ->
            printLine "I'm sorry, I couldn't determine your age. Please try again."
            !longjmp
      printLine ("Ah, " ++ Nat.toText age ++ ", a fine age.")
    runComp '(callCC p)
```


