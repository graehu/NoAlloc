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

## Pools

In this language `Pool` is a special array type. Under the hood pooled objects are reference counted. Here's an example of a simple interaction:

```python
my_pool = Pool(Int, 128)
main = Function():
    my_1 = Int(my_pool.get_recycled()) # returns index 0
    my_2 = Int(my_pool.get_recycled()) # returns index 1
    my_3 = Int(my_pool.get_recycled()) # returns index 2
    my_4 = Int(my_pool.get_recycled()) # returns index 3
    # Now I null the pointer
    my_1 = null
    # I immediately get it back
    my_5 = Int(my_pool.get_recycled()) # returns index 0

```
Internally there would be a head_index and a recycled_stack. The stack has items pushed onto it as they return to the pool. If there are no items on the recycled_stack the head_index increments and that item is returned. This approach automatically calculates the maximum needed items for any `Pool` assuming zero branch complexity.

This is useful for informing the programmer that he's overallocating for any given `Pool` or if a pool is underallocating.

This would be stored in contigious memory, with an upfront block allocation for the pool.

# Compile Time Extension

In dynamic languages it's quite common to allow members and methods to be added at runtime. For this language extension is a compile time concept. The rules are that any given type can be extended so long as it's defined within the current blob.

example:
```javascript

// simple definition
my_class = Resource():
    pass

//two legal examples
my_class.cool_member = Int(1)
my_class.cool_method = Function():
    pass

//not legal!!
Resource.bad_function():
    pass

```

The colon is used to signify an extension/type definition. The reason `bad_function` isn't legal is because `Resource` is a base type. You can't extend base types because they're not a part of the current blob.

# Resources

The compiler has extentions for Resources - `.png` being an example - which allows it to compile them into the blob with a consistent language level API. Resources are first class citizens in this language. Extentions are managed automatically by a package manager built into the compiler. The Resources introduced by extentions are types in their own right, so functionality is packed along with data.

It's also possible to compile your own Resource types as blob files to extend the compiler locally.

An example:

```python
main = Function():
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

# C++ Interop

The intention here is for .libs or .dlls to be useable from within the language. They could be treated as resources, with a translation layer. Ideally this layer would be auto generated but that may not be possible in some cases, so there should also be the option to write a "compliant wrapper" in C++ that defines functinos NoAlloc can call correctly.


# Syntax

I want the syntax to be `thing = constructor()` in every possible case. If you try to assign a new var without using a constructor it'll fail. examples:


```python
main = Function():
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

testing how fast this is. Its probably just fine for my kwyboard.