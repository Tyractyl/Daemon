# Daemon

Talon, The Tyractyl Daemon

Tyractyl Daemon is Tyractyl's server control plane, built for the rapidly changing gaming industry and designed to be
highly performant and secure. The daemon provides an HTTP API allowing you to interface directly with running server
instances, fetch server logs, generate backups, and control all aspects of the server lifecycle.

In addition, Tyractyl Daemon ships with a built-in SFTP server allowing your system to remain free of Tyractyl specific
dependencies, and allowing users to authenticate with the same credentials they would normally use to access the Panel.

## Features

- **High Performance**: Built with Go for optimal performance and low resource usage
- **Secure**: All game servers run in isolated Docker containers
- **Built-in SFTP**: No external dependencies required for file management
- **HTTP API**: Comprehensive REST API for server management
- **Real-time**: WebSocket support for live server logs and console
- **Backup Support**: Local and S3 backup storage options
- **Transfer Support**: Server transfer between nodes with zero downtime

## Documentation

* [Panel Documentation](https://pterodactyl.io/panel/1.0/getting_started.html)
* [Daemon Documentation](https://pterodactyl.io/wings/1.0/installing.html)
* [Community Guides](https://pterodactyl.io/community/about.html)

## Reporting Issues

Please use the [Tyractyl/Panel](https://github.com/Tyractyl/Panel) repository to report any issues or make
feature requests for the Daemon.
