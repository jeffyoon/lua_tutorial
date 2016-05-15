#C Code 를 Lua script 에서 사용할수 있도록 C-Lua Binding module 을 만드는것

- Lua is an extensible language
> we can extend its functionality using libraries written in other languages (Mostly C)

- Lua is an extension language
> we can extend the functionality of applications written in other languages with Lua Code.

The same API that we use to call Lua from C, for extending an application, is the API we use to call C from Lua , to implement C modules.


##C API
- The C API has a few dozen functions to read and write global variables,
  call functions and chunks, create new tables, read and write table fields, export C function to Lua etc.

- Functions of the C API are unsafe : it is the responsibility of the programmer to make sure they are called with the right arguments and in the correct context.

- This is C programming , so segmentation faults and memory corruption await the careless!


##Lua states
- The Lua interpreter does not define any C global variables: it keeps its state in a data structure called just a **Lua state**.

##Lua Stack
- All communication between Lua and C code is done through the Lua stack.
- The stack holds Lua values , and C API functions usually pop values they need from the stack and push values they produce on the stack.
- It is your responsibility to make sure the stack has enough "slots" to do what you want, and a fresh stack begin with space for 20 slots;
if you need more, use lua_checkstack:
sucess = lua_checkstack(L, 50);/* make sure there is space to push 50 values */

##Pushing values
- The C API has functions to push atomic values :
```
  void lua_pushnul(lua_State *L);
  void lua_pushboolean(lua_State *L, int bool);
  void lua_pushnumber(lua_State *L, double n);
  void lau_pushinteger(lua_State *L, ptrdiff_t i);
  void lua_pushunsigned(lua_State *L, unsigned int u);
  void lau_pushlstring(lua_State *L, const char *s, size_t len);
  void lau_pushstring(lua_State *L, const char *s);

  void lau_newtable(lua_State *L);
```

##Querying elements
- C code can reference any position in the stack with indices.
- Positive indices (from 1) count from the bottom to the stack and up.
- Negative indices (from -1) count from the top of the stack and down.

For example : -1 is always the top slot, -2 is the slot below the top and so on.
```
| -1 |  stack top      |  .. |
| -2 |  ...            |  3  |
| -3 |                 |  2  |
| .. |  stack bottom   |  1  |
```

##Type Checking
- The lau_type function is the analogue of type :
```
  int lau_type (lua_State *L, int index);
  const char * lua_typename(lua_State *L, int type);
```

##Getting atomic values out
- The API has several functions to extract atomic values from the stack (while leaving them there) :
```
  int lua_toboolean(lua_State *L, int index);
  const char *lua_tolstring(lua_State *L, int index, size_t *len);
  double lua_tonumber(lua_State *L, int index);
  ptrdiff_t lua_tointeger(lua_State *L, int index);
  unsigned int lua_tounsigned(lua_State *L, int index);
```

##Stack movement
- There are several functions to move the stack contents around, which is useful sometimes:
- Remember that all indices can be positive or negative
```
  /* index of top element */
  int lua_gettop (lua_State *L);
  /* sets the new top, popping values or pushing nils */
  void lua_settop (lua_State *L, int index);
  /* pushes a copy of the value at index */
  void lua_pushvalue(lua_State *L, int index);
  /* removes the value at index, shifting down */
  void lua_remove (lua_State *L, int index);
  /* pops the value at the top and inserts into index, shifting up */
  void lua_insert (lua_State *L, int index);
  /* pops the value at the top and inserts into index, replacing what is there */
  void lua_replace (lua_State *L, int index);
  /* copy the value at "from" to "to", replacing what is there */
  void lua_copy (lua_State *L, int from, int to);
```

##Calling a C function from Lua
- Function receives a Lua state (stack) and returns (in C) number of results (in Lua)
- Get arguments from the stack , do computation, push arguments into the stack.
```
static int l_sqrt(lua_State *L) {
  double n = luaL_checknumber(L, 1);
  lua_pushnumber(L, sqrt(n));
  return 1; /* number of results */
}
```
