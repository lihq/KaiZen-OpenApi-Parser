# P2 Update Site  Support

This directory includes a maven pom.xml file and a shell script to
support creation of a p2 update site containing OSGi bundles for KZOP
and all its dependent jar files, including JsonOverlay.

The RepreZen team uses these features to create and publish p2 bundles
for use in our [KaiZen
OpenApiEditor](https://github.com/RepreZen/KaiZen-OpenAPI-Editor)
open-source project, and our [API Studio](https://www.reprezen.com/)
commercial product. Both are designed for execution within the Eclipse
framework, which is OSGi-based.

Our update site for a given version of  KZOP is:

https://products.reprezen.com/kaizen/openapi-parser/updates/\<version\>.

Note that we do not manage a composite site covering the available
versions; you must use the site for the specific version you intend to
use.

## Building the Update Site

You can use the `build.sh` script to build the update site. For
non-RepreZen uses, you will need to customize the script if you wish
to publish your update site, or else publish your site via some other
mechanism and avoid using the `-p` option.

Here is the usage message for the script:

```
Usage: ./build.sh <options>
Builds a p2 repository for KZOP, including JsonOverlay and other dependencies, and
optionally publishes as an update site.

Options: (* marks required option)
* -k or --kzop-version - specify full version number of KZOP to package
* -j or --jovl-version - specify full version number of JsonOverlay to package
  -K or --kzop-repo - specify maven repo URL where indicated KXOP version is
     available, if is not in maven central
  -J or --jovl-repo - specify maven repo URL where indicated JsonOverlay
     version is available, if is not in maven central
  -p or --publish - publish the newly built update site
* -b or --build-scripts - speicfy location of RepreZen build scripts, including 
     the "publish" script. This is required only if "-p" is specified
  -h or --help - print this message and do nothing else

Note on repos: Both ---kzop-repo and --jovl-repo default to
http://localhost:8000, so if you already have the needed versions in
your local maven repo cache, you can set up a simple HTTP server to
serve those files and go with these defaults. But you can also explicitly
specify, e.g., staging repository URLs. 

To use the defualt repo, you can use either of the following to launch
a local web server, after positioning yourself in the root of your
local maven cacche (typically .m2/repository in your home directory):
  * python -m SimpleHTTPServer
  * python3 -m http.server
```

To build without publishing, use a command like this, with
`kaizen-openapi-parser/p2` as your working direcotry:

```
./build.sh \
    -k <kaizen-openapi-version> \
    -j <json-overlay-version>
```

## Building With Versions Not In Maven Central

The above assumes that the indicated versions of KZOP and JsonOverlay are
available in maven central. If that's not the case (e.g. if you are
buliding with versions that are present in OSS staging repos but not
yet released to central), you can follow this procedure:

1. Retrieve the needed artifacts into your local maven cache repo,
   e.g. using the `dependency:get` goal of the
   `maven-dependency-plugin` (use `-DrepoUrl` to specify a maven repo
   where the needed version is available).
2. Start an HTTP server that will serve files from your local maven
   cache repo (typically in `.m2/repository` in your home
   directory). For example, running either of the following commands
   with that as your working directory should work:
   * `python -m SimpleHTTPServer`
   * `python3 -m http.server`
3. Run the build script as shown above. If your local HTTP server is
   not listening on port 8000, you can use the `-r` option to  specify
   the root URL serving your local repo, which defaults to
   `http://localhost:8000`.

Alterantively, you can use the `--kzop-repo` (`-K`) and/or
`--jovl-repo` (`-J`) options to specify URLs for staging repositories
that contain the needed versions.

## Publishing

If you use the `-p` option, the script will publish the newly-built
update site. The script assumes that this will be accomplished using
RepreZen's standard publishing script, which is in a private GitHub
repository. If you do not work for RepreZen, you should use some other 
means to publish your update site.

If you use `-p` , you must also specify the location where the
`publish` script is located on your machine. This should be the
`build-scripts` directory in an up-to-date clone of the
`ModelSolv/BuildSupport` GitHub repo on `master` branch.

You will also need a `.ssh/config` file that defines a virtual host
named `wsw` that resolves to the machine that hosts our `products`
site, with appropriate `User` and `IdentityFile` properties.

The publish script will fully remove any existing update site for the
inidicated KZOP version, if one exists.
