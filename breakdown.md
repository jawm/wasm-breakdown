# Breakdown of a WebAssembly file.

### Text representation

This is the text representation of a program.
The program creates 2 functions, $f1, and $f2.
It then stores these two functions into a table.
Finally a 3rd function is created. This function takes an i32 arg.
This arg is used as an index to reference a function in the table.
When we call the function from the table, we must also specifiy the expected type.
In this case it is $return_i32. We defined $return_i32 as follows;
	A function which takes no parameters, and returns a single i32.

#### program.wat:

```
(module
  (table 2 anyfunc)
  (func $f1 (result i32)
    i32.const 42)
  (func $f2 (result i32)
    i32.const 13)
  (elem (i32.const 0) $f1 $f2)
  (type $return_i32 (func (result i32)))
  (func (export "callByIndex") (param $i i32) (result i32)
    get_local $i
    call_indirect $return_i32)
)
```

### Binary representation

This binary representation of our program was generated using [wabt](https://github.com/webassembly/wabt).
The command was: `wast2wasm program.wat -o program.wasm`
Below is the file as viewed using [xxd](http://linuxcommand.org/man_pages/xxd1.html)

#### program.wasm:

00000000: 0061 736d 0100 0000 010e 0360 0001 7f60  .asm.......`...`
00000010: 0001 7f60 017f 017f 0304 0300 0002 0404  ...`............
00000020: 0170 0002 070f 010b 6361 6c6c 4279 496e  .p......callByIn
00000030: 6465 7800 0209 0801 0041 000b 0200 010a  dex......A......
00000040: 1303 0400 412a 0b04 0041 0d0b 0700 2000  ....A*...A.... .
00000050: 1101 000b                                ....


### Program Breakdown:

Below is a breakdown of how the text and binary versions are equivalent
```
magic number:	6d736100	// this has been converted to big-endian
version: 		00000001	// this has been converted to big-endian

	id:				01	// types
	len:			14
	data:			0360 0001 7f60 0001 7f60 017f 017f
		count:			3
		entries:	
			form:			60 //func 0110 0000
			arg count:		0
			args:		
			ret count:		1
			ret types:		7f 	//i32

			form:			60	//func
			arg count:		0
			args:		
			ret count:		1
			ret types:		7f 	//i32

			form:			60
			arg count:		1
			args:			7f 	//i32
			ret count:		1
			ret types:		7f 	//i32

	id:				03	// function declarations	
	len:			4?
	data:			0300 0002
		count:			3
		entries:	()->i32, ()->i32, i32->i32

	id:				04 // table
	len:			4
	data:			0170 0002
		count:			1
		entries:
			element_type:	70	// anyfunc
			limits:	
				flags:		0	// maximum field not present
				initial:	2	//
				maximum:	not present, as flag is 0
	id:				07	// export
	len:			15
	data:			010b 6361 6c6c 4279 496e 6465 7800 02
		count:			1
		entries:
			field_len:		11
			field_str:		6361 6c6c 4279 496e 6465 78
			kind:			0	// function
			index:			02	// 3rd defined in the types section

	id:				09	// element
	len:			8
	data: 			01 0041 000b 0200 01
		count:			1
		entries:
			index:			0			// should be zero
			offset:			41-00-0b	// const-0-end
			num_elems:		02			// should be two
			elems:			00, 01		// should be zero, one
	
	id:				10	//code
	len:			19
	data:			03 0400 412a 0b04 0041 0d0b 0700 2000 1101 000b
		count:			3
		entries:
			size:			4			// the body size
			locals_len:		0			// number of locals
			locals:			-			// not present
			code:
				41 2a			i32.const 42
			end:			0b 			// the end byte

			size:			4
			locals_len:		0
			locals:			-
			code:			
				41 0d: 			i32.const 13
			end:			0b

			size:			07
			locals_len:		0
			locals:			-
			code:			2000 1101 00
				20 00:			get_local 0
				11 01 00		call_indirect 1
			end:			0b
```

This program didn't use all the sections in the WebAssembly spec, but is a more basic example.