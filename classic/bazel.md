startup --output_user_root=${CF_VOLUME_PATH}/bazel/output
build --build_event_json_file=${CF_VOLUME_PATH}/bazel-events.json
test --test_output=errors

build --remote_cache=http://bazel-cache.bazel-cache.svc.cluster.local
test --remote_cache=http://bazel-cache.bazel-cache.svc.cluster.local

# The following options make docker compose behave in a reasonable way when run in Codefresh. They
# are used by the compose_test rules to help translate local paths to docker daemon paths.
test --test_env=CF_VOLUME_PATH=${CF_VOLUME_PATH}
test --test_env=CF_VOLUME_PATH_DOCKER=${CF_VOLUME_PATH_DOCKER-}

# Disable color and cursor motion in the output so that things are a little
# more legible in codefresh
common --color=no --curses=no
