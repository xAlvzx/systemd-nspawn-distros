# systemd-nspawn-distros

A collection of guides for setting up and managing various Linux distributions as systemd-nspawn containers, optimized for Arch Linux hosts. These containers are lightweight, fast, and easy to manage.

---

## Available Distributions

Select a distribution below to view the detailed setup guide.

*   **[Kali Linux](./distros/kali-linux/README.md)**
*   **[Debian](./distros/debian/README.md)**
*   Arch Linux *(guide coming soon)*

---

## General Information

### What is systemd-nspawn?

`systemd-nspawn` is like `chroot` on steroids. It can be used to run a command or a whole operating system in a lightweight container. It provides better isolation than a simple `chroot` by leveraging kernel namespaces.

### Why use it?

*   **Lightweight:** Much lower overhead than full virtualization (like VirtualBox or KVM).
*   **Fast:** Containers start in seconds.
*   **Integrated:** Uses the same kernel and systemd instance as the host.
*   **Simple:** Easy to create, manage, and script.

---

## Contributing

Have a guide for a different distribution? Want to improve an existing one?

1.  Fork the repository.
2.  Create a new directory under `distros/` for your distribution (e.g., `distros/ubuntu/`).
3.  Add a `README.md` with your detailed guide.
4.  Add a link to your new guide in this main `README.md` file.
5.  Submit a pull request!

## License

This project is open source and available under the [MIT License](./LICENSE). Use the provided guides at your own risk.