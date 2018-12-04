# s3-redeploy

Node.js utility to sync files to Amazon S3 and invalidate CloudFront distributions.

# Module status: WIP until 0.1.0 release

## Usage

**Node.js >= 6.0.0 is required**

npm >= 5.2.0
```bash
$ npx s3-redeploy --bucket bucketName --pattern './**' --cwd ./folder-to-sync
```

npm < 5.2.0

```bash
$ npm i --global s3-redeploy
$ s3-redeploy --bucket bucketName --pattern './**' --cwd ./folder-to-sync
```

#### Options
```
--bucket
``` 
*Mandatory.* Name of bucket where to sync the data
```
--cwd
```
*Optional.* Path to folder to treat as current working one. Defaults to `process.cwd()`
```
--pattern
```
*Optional.* Glob pattern, applied to the `cwd` directory. Defaults to `./**`, which means all the files inside the current directory and subdirectories will be processed. **Should be passed in quotes in linux to be treated as a string.**
```
--gzip
```
*Optional.* Indicates whether the content should be gzipped. A corresponding `Content-Encoding: gzip` header added to uploading objects. If an array of extensions passed, only matching files will be gzipped, otherwise all the files are gzipped. Array should be represented as a semicolon-separated list of extensions without dots. Example: `--gzip 'html;js;css'`.
```
--profile
```
*Optional.* Name of AWS profile to be used by AWS SDK. See [AWS Docs](https://docs.aws.amazon.com/cli/latest/topic/config-vars.html). If a region is specified in credentials file under profile, it takes precedence over `--region` value
```
--region
```
*Optional.* Name of the region, where to apply the changes.
```
--cf-dist-id X
```
*Optional.* Id of CloudFront distribution to invalidate.
```
--cf-inv-paths
```
*Optional.* Semicolon-separated list of paths to invalidate in CloudFront. Example: '/images/image1.jpg;/assets/\*'. Default value is '/\*'.
```
--ignore-map
```
*Optional.* Dictionary of files and correspondent hashes will be ignored upon difference computation. This is helpful if state of S3 bucket was changed manually (not through s3-redeploy script) but dictionary remained the same. In this case, the dictionary state will be omitted during computation and at the same time **a new dictionary will be computed and uploaded to S3** so it could be used in further invocation
```
--no-map
```
*Optional.* Use this flag to store and use no file hashes dictionary at all. Each script invocation will seek through the whole bucket and gather ETags. **If bucket already contains a dictionary file, it will remain as is but won't be used**
```
--no-rm
```
*Optional.* By default all the removed locally files will be also removed from S3 during sync. Use this flag to override default behavior and upload new files / update changed ones only. No files will be removed from S3. At the same time, the file hashes map will be updated to mirror relevant S3 bucket state properly.
```
--concurrency X
```
*Optional.* Parameter sets the maximum possible amount of network / file system operations to be ran in parallel. Defaults to 5. *Note:* it is safe to run file system operations in parallel due to streams usage
```
--file-name
```
*Optional.* Utility uploads a file, containing md5 hashes, upon folder sync. This file is used during the sync operation and lets to minimize amount of network requests and computations. Defaults to `_s3-rd.<bucket name>.json`
```
--cache X
```
*Optional.* Sets `Cache-Control: max-age=X` for uploaded files. Must be passed in seconds. By default nothing is set.

## Backgorund

Package provides an ability to sync a local folder with an Amazon S3 bucket and create an invalidation for a CloudFront distribution. Extremely helpful if you use S3 bucket as a hosting for your website.

The module itself may be both executed as a script from the command line and imported as a module into the application. The idea was inspired by [s3-deploy](https://www.npmjs.com/package/s3-deploy) but another approach to work out the sync process was taken. The set of functionality is also slightly different.

#### How it works

In order to decrease amount of S3 invocations and increase processing speed, MD5 hashes dictionary is built for all the local files. The dictionary is uploaded along with other files and is used during further updates. Instead of querying S3 for ETags, the S3-stored dictionary is compared with local one. Thus, only changed files are updated.

**IMPORTANT:** If you change the state of bucket manually, the contents of the dictionary will not be updated. Thus, perform all the bucket update operations through the script or consider manual dictionary removal / `--ignore-map` flag usage, which will let the dictionary to be computed again and stored on the next script invocation.

## Tests

Describe and show how to run the tests with code examples.

## Contributors

Let people know how they can dive into the project, include important links to things like issue trackers, irc, twitter accounts if applicable.

## License

MIT


## TODO:
* fix versions in package.json
* precommit hook
* unit tests
* add Travis and coverall + badges
* use maps instead of objects
* add verbose / silent flags and improve logging
* build redirect objects
* check on Windows

Additional things to consider:
* check what could be done with versions and prefixes for S3 objects
* take a look at ACL:private for hash map
