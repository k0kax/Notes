
lua的执行流程：
```lua
编写Lua源码（.lua文件） → 2. 编译阶段（luac处理） → 3. 生成字节码Chunk（.luac文件） → 4. 虚拟机加载字节码 → 5. 执行字节码输出结果
```
![[attachement/Pasted image 20251227201123.png]]

luac是编译器也是反编译器