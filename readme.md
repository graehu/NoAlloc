# NoAlloc

The language where you can't dynamically allocate anything!

# Language Goals

* Highly memory oriented
  * No fragmentation
  * No leaks
  * No corruptions
  * Fast instatiations
  * Stack driven
* Resources are first class citizens
  * Saved into the binaries
  * Classes with functions


# Memory Management
 
Objects are managed as loaded binary blobs with pre-allocated pools of internal objects. These blobs contain everything needed to execute their code. Functions would be copy constructed onto the main program stack.

When loading and unloading blobs the language is making heap allocations to seat them in. To avoid fragmentation over long periods of run time, when a blob is unloaded all blobs will need to be reseated. Meaning all pointers will need to be fixed up too.

Pointers store a reference to the blobs header. On a reseat all blobs would calculate their delta displacement in memory, then store that in their headers. Next all pointers are itterated to update based on the memory delta. Finally the blob is reseated.

You can mark sections of your program to be built into blobs that are loaded as `.blobs`.
They're also a type of resource, but their API is defined by you.

# Resources

The compiler has extentions for Resources - `.png` being an example - which allows it to compile them into the blob with a consistent language level API. Resources are first class citizens in this language. Extentions are managed automatically by a package manager built into the compiler. The Resources introduced by extentions are types in their own right, so functionality is packed along with data.

It's also possible to compile your own Resource types as blob files to extend the compiler locally.

An example:

```python
main = function():
    # This is a declaration, with an inline construction.
    # The compiler would find this and know to compile the texture into the binary.
    # "texture" becomes a pointer to the resource.
    # Resources are only compiled into the blob once unless otherwised specified  
    texture = Resource("x:/my_art/albedo.png")

    # Possible syntax to ignore previously loaded resources and compile a new version. 
    texture2 = Resource("x:/my_art/albedo.png", true)
    # get_num_pixels defined by .png object
    pixel_count = int(texture2.get_num_pixels())
```

Above you can see that there's no explicit type. In this case the type is `.png` and the compiler would need to have an extention installed in order to compile `.png`. 

# Syntax

I want the syntax to be `thing = constructor()` in every possible case. If you try to assign a new var without using a constructor it'll fail. examples:


```python
main = function():
    # this is fine!
    texture = Resource("x:/my_art/albedo.png")
    pixel_count = int(texture.get_num_pixels())
    # this is also fine!
    pixel_count = 2
    # this is NOT fine.
    new_count = 10
    # this is!
    new_count_correct = int(10)
```

this might even extend to functions and classes:

```python
main = Function():
    pass
    # things!
```
## class example

```python
Car = Resource():
    #funcs
    Car = Function():
        pass
    Car = Function():
        pass
    cool_method = Function():
        pass
    
    cool_member = Int(0)
    cool_array_member = Pool(Int, 10)
    cool_string = String(32)
    cool_string2 = String(32, "fun constructor!")
    #not sure about this!
    cool_map_member = Map(String(32), Int, 16)
```

So something like

```python
my_game = Resource("my_game.blob")

main = Function():
    run_game = bool(true)
    while run_game:
        run_game = my_game.run()
    my_game.unload()
    
```

# Pool

In this language `Pool` is a special array type. All objects and types have a `Pool` API: 
* `.reserved_from_pool()`
  * returns true if it's in use.
* `.return_to_pool()`
  * returns the object to the `Pool`.
* `something.`

The pool has a `get_pooled()` function that returns.

TODO: this section is poorly defined and should actually be implemented as a reference count. But basically, the pool has `get_pooled()` and that object will be reserved UNTIL it runs out of references. At which point it will be returned to the pool to be used.

TODO TODO: Reword this.

# Possible Issues

- Paging for huge allocations might be awkward.
- Consistent packing as the compiler adds features could be an issue.
- Managing changing data/binary layouts of types
    - Especially in the resources because they're managed by plugin!
    - will need a consistent header for ALL resources that can be error checked.

# Unfounded ideas!
- no auto objects on the stack! i.e. my_game = Resource("my_game.blob") in function scope.
- use : to signify an extention/implementation
```python
#class defs!
my_class = Resource("something.png"):
    # junk

my_extra_args = Resource("something.png": duplicates = int(2), tint=Color(0.5,0.5,0.5)):
    #more junk!
```