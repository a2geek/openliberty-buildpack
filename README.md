# Open Liberty Buildpack

Experimental Cloud Foundry for Open Liberty.

# Building the Buildpack

> Note: the buildpack currently only works as a static buildpack.

```
$ buildpack-packager build -cached -any-stack
```

Upload the resulting buildpack as usual.

# Deployment

> The package structure is subject to change.

Package the deployment in a Zip file. Contents are as follows:

```
apps/
  WAR/EAR files
config/
  server.xml
```

`cf push` the app as usual.

# Resources

* [About Buildpacks](https://docs.cloudfoundry.org/buildpacks/understand-buildpacks.html)
* [Creating Custom Buildpacks](https://docs.cloudfoundry.org/buildpacks/custom.html)
* [Buildpack Packager](https://github.com/cloudfoundry/libbuildpack/tree/master/packager)
* [Open Liberty Docs](https://openliberty.io/docs/22.0.0.4/overview.html)

# See Also 

* [Paketo Buildpacks](https://github.com/paketo-buildpacks/liberty)
