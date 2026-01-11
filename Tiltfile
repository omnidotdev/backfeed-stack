load("ext://git_resource", "git_checkout")
load("ext://color", "color")


# load configuration file
services = read_yaml("services.yaml").get("services")


# bootstrap env vars
for service in services:
    values = service.values()[0]

    if "env" in values:
        for var in values["env"]:
            key = var.keys()[0]
            val = var.values()[0]
            print(color.yellow("Setting `%s`..." % key))
            os.putenv(key, val)


# bootstrap project services
for service in services:
    name = service.keys()[0]
    values = service.values()[0]

    # handle metarepos - auto-discover services in {path}/services/*/Tiltfile
    if "metarepo" in values and values["metarepo"]:
        base_path = values.get("path", "services/%s" % name)
        repo = values.get("repo")

        # expand ~ to home directory
        if base_path.startswith("~"):
            base_path = os.path.expandvars(base_path.replace("~", "$HOME"))

        # checkout only if path does not already exist
        if repo and not os.path.exists(base_path):
            print(color.yellow("%s does not exist, cloning..." % base_path))
            git_checkout(repo, base_path)
        elif os.path.exists(base_path):
            print(color.green("%s already exists" % base_path))

        # auto-discover services using shell (listdir doesn't work for external paths)
        services_dir = "%s/services" % base_path
        if os.path.exists(services_dir):
            sub_services = str(local("ls %s" % services_dir, quiet=True)).strip().split("\n")
            for sub_service in sub_services:
                if sub_service:  # skip empty strings
                    sub_path = "%s/%s" % (services_dir, sub_service)
                    tiltfile_path = "%s/Tiltfile" % sub_path
                    if os.path.exists(tiltfile_path):
                        print(color.green("     Loading Tiltfile for %s..." % sub_service))
                        include(tiltfile_path)
        continue

    repo = values["repo"]
    path = "services/%s" % name

    # use custom path if present
    if "path" in values:
        path = values["path"]

    # checkout only if path does not already exist
    # ! NB: without this path existence check, data loss may occur due to overwriting. Be very careful if disabling this.
    if not os.path.exists(path):
        print(color.yellow("%s does not exist, cloning..." % path))
        git_checkout(repo, path)
    else:
        print(color.green("%s already exists" % path))

    # load service Tiltfile if present
    if os.path.exists("%s/%s" % (path, "Tiltfile")):
        print(color.green("     Loading Tiltfile for %s..." % name))
        include(os.path.join(path, "Tiltfile"))
