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
            print(color.yellow("🗺️ Setting `%s`..." % key))
            os.putenv(key, val)


# bootstrap project services
for service in services:
    name = service.keys()[0]
    values = service.values()[0]
    repo = values["repo"]

    path = "services/%s" % name

    # use custom path if present
    if "path" in values:
        path = values["path"]

    local_resource("dev-%s" % name, serve_cmd="cd %s && bun dev" % path, labels=[name])

    # checkout only if path does not already exist
    # ! NB: without this path existence check, data loss may occur due to overwriting. Be very careful if disabling this.
    if not os.path.exists(path):
        print(color.yellow("💡 %s does not exist, cloning..." % path))
        git_checkout(repo, path)
    else:
        print(color.green("✅ %s already exists" % path))

    # load service Tiltfile if present
    if os.path.exists("%s/%s" % (path, "Tiltfile")):
        print(color.green("     ❇️ Loading Tiltfile for %s..." % name))
        include(os.path.join(path, "Tiltfile"))
