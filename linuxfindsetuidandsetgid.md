# 0. Introduction/Setup.
On Linux systems, executables have special bits that can be set called setuid and setgid (set user id, set group id). Executables with the setuid bit are capable of running as their owner rather than the current user. Executables with the setgid bit are the same but with the users' groups. This can be a serious security concern as one user may be able to gain the privileges of another, potentially even root privileges. This can also be benign and even required for some programs or utilities to function. It is only an issue if there are vulnerabilities in an executable with setuid or setgid.

This short tutorial gives commands which are useful for auditing a Linux system for these types of executables. This can be used to conduct a security audit or a penetration test on a system. Unusual executables or executables that can run arbitrary code or shell commands are cause for concern when reviewing your results. Note that these commands may take a while to run on a system with many files or a slow hard drive.

# 1. The commands.
The following command finds executables with either the setuid or setgid bit set which are owned by the root user. Errors are directed to /dev/null (basically sent nowhere and not displayed) to avoid flooding one with unnecessary information.
> find / -user root \( -perm 4000 -o -perm -2000 \) 2>/dev/null

The following command finds executables with either the setuid or setgid bit set which are owned by any user. Errors are directed to /dev/null (basically sent nowhere and not displayed) to avoid flooding one with unnecessary information.
> find / \( -perm 4000 -o -perm -2000 \) 2>/dev/null

These commands may be modified as one sees fit. For instance, replacing `/` (the root of the filesystem) with a specific directory to examine.
