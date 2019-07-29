# WasmerSharp

.NET Bindings for the Wasmer Runtime.

This binds Wasmer at version ab5f28851a676f9d3672f41d1608e34ddab470ff

# Install

The Wasmer bindings are a .NET Standard library, and they will need
the Wasmer C runtime to be installed somewhere accessible in your
system (either the same directory as the DLL, or in a location
accessible to the dynamic linker).

To obtain the native Wasmer C runtime, you can build Wasmer like this:

```
cargo build -p wasmer-runtime-c-api
```

And then copy the `target/debug/libwasmer_runtime_c_api.dylib` library
to the destination.

# Examples

```
using WasmerSharp

        // This method is invoked by the WebAssembly code.
	public static void Print (InstanceContext ctx, int ptr, int len)
	{
		Console.WriteLine (".NET Print called");
		var memoryBase = ctx.GetMemory (0).Data;
		unsafe {
			var str = System.Text.Encoding.UTF8.GetString ((byte*)memoryBase + ptr, len);

			Console.WriteLine ("Received this utf string: [{0}]", str);
		}
	}
	
	public static void Main (string [] args)
	{
		//
		// Creates the imports for the instance
		//
		var func = new Import ("env", "_print_str", new ImportFunction ((Action<InstanceContext,int,int>) (Print)));
		var memory = new Import ("env", "memory", Memory.Create (minPages: 256, maxPages: 256));
		var global = new Import ("env", "__memory_base", new Global (1024, false));

		var wasm = File.ReadAllBytes ("target.wasm");

		//
		// Create an instance from the wasm bytes, the declared func, the memory we created and the global we have
		//
		var instance = new Instance (wasm, func, memory, global);

		//
		// Call the method defined in webassembly
		//
		var ret = instance.Call ("_hello_wasm");
		if (ret == null)
			Console.WriteLine ("error calling the method: " + instance.LastError);
		else
			Console.WriteLine ("__hello_wasm returned: " + ret);
	}
```

# LICENSE

This is licensed under the MIT License terms.