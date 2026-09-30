FROM busybox
RUN --mount=type=secret,id=probe test "$(cat /run/secrets/probe)" = ok
RUN uname -m > /arch
