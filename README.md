nbqueue
=======

super simple queue module for executing async functions in node.js. nbqueue was created as a module for [nodebee](//github.com/kvonflotow/nodebee/), a pure javascript database.

## uses

#### limiting the number of functions being executed

when accessing large amounts of files asynchronously, bad things can happen. for example, take a directory with 20,000 files in it and try accessing them all at once. good luck!

with nbqueue you can set the maxProcesses to a sane amount, like 100, and it will execute a maximum of 100 functions at a time. functions added after the limit has been reached will execute as the previous functions finish up.

this is also crucial for maintaining a sane level of forked child processes.

```javascript
var fs = require( 'fs' )

var Queue = require( 'nbqueue' )

fs.readdir( directory, function ( err, files )
	{
		// set maxProcesses to 100
		var queue = new Queue( 100 ),

			data = {}

		files.forEach( function ( file )
			{
				queue.add( function ( next )
					{
						fs.readFile( file, function ( err, data )
							{
								if ( err )
								{
									return next( err )
								}

								// ... do something ...
								data[ file ] = data

								next()
							}
						)
					}
				)
			}
		)

		queue.done( function ( err )
			{
				if ( err )
				{
					// if there were errors, they will be in an array
					console.log( err )
				}

				// data is fully populated with filenames and contents
				console.log( data )
			}
		)
	}
)
```

#### executing async functions in order

setting maxProcesses to 1 will ensure only 1 function can be executed at a time.
