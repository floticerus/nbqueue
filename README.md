nbqueue
=======

super simple queue module for executing async functions in node.js. nbqueue was created as a module for [nodebee](//github.com/kvonflotow/nodebee/), a pure javascript database.

## installation
`npm install nbqueue`

## uses

#### limiting the number of functions being executed

when accessing large amounts of files asynchronously, bad things can happen. for example, take a directory with 20,000 files in it and try accessing them all at once.

with nbqueue you can set the maxProcesses to a sane amount, like 100, and it will execute a maximum of 100 functions at a time. functions added after the limit has been reached will execute as the previous functions finish up.

this is also crucial for maintaining a sane level of forked child processes.

```javascript
var fs = require( 'fs' )

var Queue = require( 'nbqueue' )

fs.readdir( directory, function ( err, files )
	{
		// set maxProcesses to 100
		var queue = new Queue( 100 )

		files.forEach( function ( file )
			{
				queue.add( function ( next )
					{
						fs.readFile( file, function ( err, data )
							{
								next( err, data )
							}
						)

						// because next takes the same arguments as the readFile callback, you could do
						// fs.readFile( file, next )
					}
				)
			}
		)

		queue.done( function ( err, results )
			{
				if ( err )
				{
					// if there were errors, they will be in an array
					console.log( err )
				}

				// results contains an array of the results passed to next()
				console.log( results )
			}
		)
	}
)
```

#### executing async functions in order

setting maxProcesses to 1 will ensure only 1 function can be executed at a time.
