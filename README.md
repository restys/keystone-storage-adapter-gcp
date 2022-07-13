
This adapter only works with Keystone Classic (Keystone v4 and below).

# GCP-based storage adapter for KeystoneJS

[![Build Status](https://travis-ci.org/keystonejs/keystone-storage-adapter-s3.svg?branch=master)](https://travis-ci.org/keystonejs/keystone-storage-adapter-s3)

This adapter is designed to replace the existing `GCPFile` field in KeystoneJS using the new storage API.

## Usage

Configure the storage adapter:

```js
var storage = new keystone.Storage({
  adapter: require('keystone-storage-adapter-gcp'),
  s3: {
    bucket: 'mybucket', // required; defaults to process.env.GCP_BUCKET
    path: '/profilepics', // optional; defaults to "/"
    publicUrl: "https://xxxxxx.cloudfront.net", // optional; sets a custom domain for public urls - see below for details
    uploadParams: { // optional; add GCP upload params; see below for details
      ACL: 'public-read',
    },
  },
  schema: {
    bucket: true, // optional; store the bucket the file was uploaded to in your db
    etag: true, // optional; store the etag for the resource
    path: true, // optional; store the path of the file in your db
    url: true, // optional; generate & store a public URL
  },
});
```

Then use it as the storage provider for a File field:

```js
File.add({
  name: { type: String },
  file: { type: Types.File, storage: storage },
});
```

### Options:

The adapter requires an additional `gcp` field added to the storage options. It accepts the following values:

- **bucket**: *(required)* GCP bucket to upload files to. Bucket must be created before it can be used. Configure your bucket through the GCP console

- **path**: Storage path inside the bucket. By default uploaded files will be stored in the root of the bucket. You can override this by specifying a base path here. Base path must be absolute, for example '/images/profilepics'.

- **publicUrl**: Provide a custom domain to serve your S3 files from. This is useful if you are storing in S3 but reading through a CDN like Cloudfront. Provide either the domain as a `string` eg. `publicUrl: "https://xxxxxx.cloudfront.net"` or a function which takes a single parameter `file` and return the full public url to the file.

Example with function:

```
publicUrl: (file) => `https://xxxxxx.cloudfront.net${file.path}/${file.filename}`;
```

- **generateFilename**: A function that accepts a file, a parameter and a callback to generate a strong pseudo-random 16 byte filename.

```js
generateFilename: (file, param, cb) => { cb(null, file.filename); }
```

### Schema

The GCP adapter supports all the standard Keystone file schema fields. It also supports storing the following values per-file:

- **bucket**: The bucket for the file to be stored in the database. If this is present when reading or deleting files, it will be used instead of looking at the adapter configuration. The effect of this is that you can have some (eg, old) files in your collection stored in different buckets.

- **path**: The path within the bucket. If this is present when reading or deleting files, it will be used instead of looking at the adapter configuration. The effect of this is that you can have some (eg, old) files in your collection stored in different paths inside your bucket.

The main use for both of these values is to allow slow data migrations. If you *don't* store these values you can arguably migrate your data more easily - just move it all, then reconfigure and restart your server.

- **etag**: The etag of the stored item. This is equal to the MD5 sum of the file content.

- **url**: The absolute URL path of the file located on s3.


# Change Log

## v2.0.0

### Additions

- **publicUrl**: You can now customise the public url by passing either a domain name as a string (eg. `{ publicUrl: "https://xxxxxx.cloudfront.net" }`) or by passing a function which takes the `file` object and returns a the url as a string.
```js
{ publicUrl: file => `https://xxxxxx.cloudfront.net${file.path}/${file.filename}` }
```

### Other

- **path**: The requirement for `path` to have a **leading slash** has been removed. The previous implementation failed to catch this miss-configuration and Knox helpfully made the file uploads work anyway. This has lead to a situation where it is possible/likely that there are existing installations where a miss-configured path is stored in the database. To avoid breaking these installs we now handle adding or removing the leading slash as required.

# License

Licensed under the standard MIT license. See [LICENSE](license).
