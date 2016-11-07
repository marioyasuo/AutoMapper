AutoMapper uses MyGet to publish development builds based on the master branch. This means that the MyGet build sometimes contains fixes that are not available in the current NuGet package. Please try the latest MyGet build before reporting issues, in case your issue has already been fixed but not released.

The AutoMapper MyGet gallery is available at http://myget.org/gallery/automapperdev.

# Installing the Package
If you want to install the latest MyGet package into a project, you can use the following command:

```
Install-Package AutoMapper -Version 5.2.0-alpha-01235 -Source https://www.myget.org/F/automapperdev/api/v3/index.json
```

`5.2.0-alpha-01235` should be replaced with the version number of the latest package, which can be found on the MyGet gallery.
