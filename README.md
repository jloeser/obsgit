# obsgit
Simple bridge between Open Build Server and git.

This tools can be used to export a project stored in OBS into a local
git repository, and later imported from git to the same (or different)
OBS server.


# Configuration
`obsgit` requires a configuration file to adjust the parameters of the
different OBS services, the remote storage for big files, and some
configuration of the layout of the local git repository.

You can generate a default configuration file with:

```
obsgit create-config
```

This command accept parameters to adjust the configuration, but you
can also edit the generated file to set the passwords and the
different URL for the APIs:

```
[export]
# API URL for the build system where we will export from (to git)
url = https://api.opensuse.org
# Login credentials
username = user
password = password
# What to do when obsgit read a linked package:
# - always: always expand the _link, downloading the expanded source
# - never: never expand, download only the _link file. If link is
#     pointing to a different project, generate an error for this package
# - auto: expand the link only if point to a different project
link = never

[import]
# API URL for the build system where we will upload the project (from git)
url = https://api.opensuse.org
username = aplanas
password = opensuse2013 

[git]
# Directory name used to store all the packages. If missing, the packages
# will be stored under the git repository
prefix = packages

[storage]
# Type of repository where to store large binary files
# - lfs: use git-lfs protocol (require git-lfs binary)
# - obs: use OBS / IBS to store the large files
type = lfs
# (obs) API URL for the build system to store files
# url = https://api.opensuse.org
# username = aplanas
# password = opensuse2013
# (obs) Repository and package where to store the files
# storage = home:user:storage/files
```


# Export from OBS to git

The `export` sub-command can be used to read all the metadata of a OBS
project, the list of packages and the content, and download them in a
local git repository. This information is organized with to goals in
mind. One is to collect all the information required to re-publish the
project and packages in a different OBS instance, and the other one is
to delegate into git the management of the package assets (change log,
spec files, patches, etc).

To export a full project:

```
obsgit export openSUSE:Factory ~/Project/factory-git
```

If required, this command will initialize the local git repository
given as a second parameter, and using the credentials from the
configuration file, download all the metadata and packages from the
project.

We can also export a single package:

```
obsgit export --package gcc openSUSE:Factory ~/Project/factory-git
```

If we are using the `lfs` extension of git, the export will create a
`.gitattributes` file that reference all the detected binary
files. You can use the `git lfs` commands to add or remove tracked
files, add them to the index and do the commit.

When the storage is configured to use `obs`, the binary files are
uploaded into the storage server, and tracked in the
`<package>/.obs/files` metadata file.


# Import from git to OBS

We can re-create the original project that we exported from OBS to git
into a different build service. To do that we can use the `import`
sub-command:

```
obsgit import ~/Project/factory-git home:user:import
```

In the same way, we can use the `--package` parameter to restrict the
import to a single package.
