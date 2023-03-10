# @kogs/utils &middot; ![tests status](https://github.com/Kruithne/kogs-utils/actions/workflows/github-actions-test.yml/badge.svg) ![typescript](https://img.shields.io/badge/language-typescript-blue) [![license badge](https://img.shields.io/github/license/Kruithne/kogs-utils?color=blue)](LICENSE) ![npm version](https://img.shields.io/npm/v/@kogs/utils?color=blue)

`@kogs/utils` is a [Node.js](https://nodejs.org/en/) package that provides a collection of utility functions.

- Provides ["pure functions"](https://en.wikipedia.org/wiki/Pure_function).
- Full [TypeScript](https://www.typescriptlang.org/) definitions.
- Zero dependencies.
- Designed to be [tree-shakeable.](https://en.wikipedia.org/wiki/Tree_shaking)

## Installation
```bash
npm install @kogs/utils
```

## Usage
```js
// Import functions individually.
import { someFunction } from '@kogs/utils';

// Or import the entire module.
import utils from '@kogs/utils';
```

## API

- [`copy`](#copy) - Copy a file or directory recursively.
- [`copySync`](#copysync) - Synchronous version of `copy`.
- [`collectFiles`](#collectfiles) - Collect all files in a directory recursively.
- [`arrayToStream`](#arraytostream) - Convert an array of values to a readable stream.
- [`streamToArray`](#streamtoarray) - Convert a readable stream to an array of values.
- [`streamToBuffer`](#streamtobuffer) - Convert a readable stream to a `Buffer`.
- [`filterStream`](#filterstream) - Create a transform stream that filters stream data.
- [`mergeStreams`](#mergestreams) - Merge multiple readable streams into a single stream.

### copy
`copy(src: string, dest: string): Promise<void>`

This method accepts a source path and a destination path and returns a promise that resolves when the copy operation has completed.

- If given a directory, the directory and all of its contents will be copied recursively to the new location.
- If given a directory, but a file exists at the destination, the file will be unlinked and replaced with the directory.
- If given a file, but a directory exists at the destination, the directory will be recursively deleted and replaced with the file.

```js
await copy('/path/to/src', '/path/to/dest');
```

The default behavior of overwriting can be controlled by passing an options object as the third argument with the `overwrite` property set to `true`, `false`, `always`, `never` or `newer`.

- `true` or `always` - Overwrite the destination if it already exists.
- `false` or `never` - Do not overwrite the destination if it already exists.
- `newer` - Only overwrite the destination tgat already exists if the source is newer.

```js
await copy('/path/to/src', '/path/to/dest', { overwrite: 'newer' });
```


### copySync
`copySync(src: string, dest: string): void`

This method accepts a source path and a destination path and returns when the copy operation has completed.

If given a directory, the directory and all of its contents will be copied recursively.

- If given a directory, the directory and all of its contents will be copied recursively to the new location.
- If given a directory, but a file exists at the destination, the file will be unlinked and replaced with the directory.
- If given a file, but a directory exists at the destination, the directory will be recursively deleted and replaced with the file.

```js
copySync('/path/to/src', '/path/to/dest');
```

The default behavior of overwriting can be controlled by passing an options object as the third argument with the `overwrite` property set to `true`, `false`, `always`, `never` or `newer`.

- `true` or `always` - Overwrite the destination if it already exists.
- `false` or `never` - Do not overwrite the destination if it already exists.
- `newer` - Only overwrite the destination tgat already exists if the source is newer.

```js
copySync('/path/to/src', '/path/to/dest', { overwrite: 'newer' });
```

### collectFiles
`collectFiles(dir: string, filter?: FileFilter): Promise<string[]>`

```js
// Relevant types:
type FileFilter = (entryPath: string) => boolean;
```

This method accepts a directory path and returns a promise that resolves with an array of file paths recursively contained within the directory.

```js
const files = await collectFiles('/path/to/dir');
// files[0] = '/path/to/dir/file1.txt'
// files[1] = '/path/to/dir/file2.log'
```

If `filter` is defined, it will be used to filter the files returned by the method. The `filter` function will be passed the combined path of the directory and file name, and should return `true` if the file should be included in the result.

```js
const files = await collectFiles('/path/to/dir', e => e.endsWith('.txt'));
// files[0] = '/path/to/dir/file1.txt'
```

### errorClass
`errorClass(name: string): new() => Error`

This method accepts a name and returns a new error class that can be used to create new errors.

```js
const MyCustomError = errorClass('MyCustomError');
throw new MyCustomError('Something went wrong!');

// error.name === 'MyCustomError'
// error.message === 'Something went wrong!'
```

The purpose of this factory is to reduce the amount of boilerplate required to create custom errors.

```js
// Common boilerplate:
class MyCustomError extends Error {
	constructor(message) {
		super(message);
		this.name = 'MyCustomError';
	}
}

// Equivalent to:
const MyCustomError = errorClass('MyCustomError');
```

### arrayToStream
`arrayToStream(input: Array<ReadableChunk>, objectMode: boolean = true): stream.Readable`

```js
// Relevant types:
type ReadableChunk = string | Buffer | Uint8Array | null;
```

This method accepts an array of values and returns a readable stream that emits each value in the array.

If `objectMode` is not defined, it will default to `true` if the first element in the `input` array is a non-null object, otherwise it will default to `false`. See the [Node.js documentation](https://nodejs.org/api/stream.html#object-mode) for more information on object mode.

```js
// Example usage.
import { arrayToStream } from '@kogs/utils';

const stream = arrayToStream(['foo', 'bar', 'baz']);
// stream.read() ==== 'foo'
```

### streamToArray
`streamToArray(input: stream.Readable): Promise<Array<ReadableChunk>>`

```js
// Relevant types:
type ReadableChunk = string | Buffer | Uint8Array | null;
```

This method accepts a readable stream and returns a promise that resolves with an array of values emitted by the stream.

```js
// Example usage.
import { streamToArray } from '@kogs/utils';

const stream = ...; // Readable stream containing 'foo', 'bar', 'baz'.
const result = await streamToArray(stream);
// result === ['foo', 'bar', 'baz']
```

### streamToBuffer
`streamToBuffer(input: stream.Readable): Promise<Buffer>`

This method accepts a readable stream and returns a promise that resolves with a `Buffer` containing the data emitted by the stream.

```js
// Example usage.
import { streamToBuffer } from '@kogs/utils';

const stream = ...; // Readable stream containing 'foo', 'bar', 'baz'.
const result = await streamToBuffer(stream);
// `result` is a Buffer[9] containing 'foobarbaz'.
```

### filterStream
`filterStream(fn: StreamFilter, objectMode: boolean = true): stream.Transform`

```js
// Relevant types:
type StreamFilter = (chunk: ReadableChunk) => Promise<boolean>;
```

This method accepts a filter function and returns a transform stream that filters the data emitted by a stream.

```js
// Example usage.
import { filterStream } from '@kogs/utils';

const stream = ...; // Readable stream containing 'foo', 'bar', 'baz'.
const filtered = stream.pipe(filterStream(chunk => chunk !== 'bar'));

// `filtered` will emit 'foo' and 'baz'.
```

### mergeStreams
`mergeStreams(...streams: Array<stream.Readable>): Promise<stream.PassThrough>`

This method accepts a variable number of readable streams and returns a promise that resolves with a `PassThrough` stream that merges the data emitted by the input streams.

See the [Node.js documentation](https://nodejs.org/api/stream.html#class-streampassthrough) for more information on `PassThrough` streams.

```js
// Example usage.
import { mergeStreams } from '@kogs/utils';

const stream1 = ...; // Readable stream containing 'foo', 'bar', 'baz'.
const stream2 = ...; // Readable stream containing 'qux', 'quux', 'corge'.

const merged = await mergeStreams(stream1, stream2);
// `merged` will emit 'foo', 'bar', 'baz', 'qux', 'quux', 'corge'.
```

## What is `@kogs`?
`@kogs` is a collection of packages that I've written to consolidate the code I often reuse across my projects with the following goals in mind:

- Consistent API.
- Minimal dependencies.
- Full TypeScript definitions.
- Avoid feature creep.
- ES6+ syntax.

All of the packages in the `@kogs` collection can be found [on npm under the `@kogs` scope.](https://www.npmjs.com/org/kogs)

## Contributing / Feedback / Issues
Feedback, bug reports and contributions are welcome. Please use the [GitHub issue tracker](https://github.com/Kruithne/kogs-utils/issues) and follow the guidelines found in the [CONTRIBUTING](CONTRIBUTING.md) file.

## License
The code in this repository is licensed under the ISC license. See the [LICENSE](LICENSE) file for more information.