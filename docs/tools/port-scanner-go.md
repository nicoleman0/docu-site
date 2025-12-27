# Building a Port Scanner in Go

*A lightning fast port scanner written in Go.*

[GitHub Repo](https://github.com/nicoleman0/portscanner-go)

As a security enthusiast, I've found that understanding network reconnaissance is foundational. To deepen my knowledge of both Go and network security, I built **portscanner-go**: a lightweight, concurrent TCP port scanner designed for speed and simplicity.

---

## Why Build a Port Scanner?

Port scanners are essential tools in penetration testing and security auditing. They help identify which services are running on a target system, providing the first step in vulnerability assessment. While tools like Nmap are industry standards, building my own scanner helped me understand:

- TCP connection mechanisms and network timeouts
- Concurrent programming patterns in Go
- Service fingerprinting techniques
- Practical network security concepts

---

## Project Architecture

The scanner is organized into three main components:

### 1. **Core Scanner Logic** (`internal/scanner/`)
The heart of the application uses a worker pool pattern to perform concurrent TCP connect scans:

```go
func ScanHostPorts(host string, ports []int, timeout time.Duration, workers int, probe bool) []Result
```

Key features:
- **Worker Pool**: Configurable number of goroutines (default: 500) to balance speed and resource usage
- **TCP Connect Scan**: Uses standard `net.DialTimeout()` - no raw sockets or root privileges required
- **Latency Tracking**: Measures connection time to identify fast vs. slow responses
- **Service Fingerprinting**: Optional probing to identify services beyond just port numbers

### 2. **Service Detection**
When the `-service` flag is enabled, the scanner attempts to identify running services through multiple techniques:

**Banner Grabbing**: Reads initial data sent by the service (SSH, SMTP, FTP, etc.)

```go
func fingerprintService(host string, port int, conn net.Conn, timeout time.Duration) (string, string, string)
```

**HTTP Probing**: Sends HTTP requests to common web ports to detect web servers and extract Server headers

**TLS Handshake**: Performs TLS negotiation on HTTPS ports to extract certificate information (CN, issuer, ALPN)

The scanner recognizes common protocols including SSH, HTTP/HTTPS, SMTP, FTP, POP3, IMAP, and many others.

### 3. **Output Formatting** (`internal/output/`)
Results are displayed in a clean, colorized table format when outputting to a terminal:

- **Color-coded states**: Green for open ports, red for closed
- **Latency indicators**: Green (<5ms), yellow (<50ms), red (>50ms)
- **Service information**: Shows detected service names, banners, and fingerprints
- **JSON support**: Optional `-json` flag for programmatic consumption

---

## Key Features

### Flexible Port Selection
The scanner supports multiple ways to specify ports:

- `top:N` - Scan the N most common ports (default: top:1000)
- Ranges: `1-1024`
- Specific ports: `80,443,8080`
- Combinations: `1-1024,8080,8443`

### Multiple Target Support
Specify targets as:
- Single hosts: `192.168.1.1`
- Comma-separated: `192.168.1.1,192.168.1.2`
- CIDR notation: `192.168.1.0/24` (IPv4 only)

### Performance Tuning
- `-workers`: Adjust concurrency level (default: 500)
- `-timeout`: Set connection timeout per port (default: 500ms)
- Balance between speed and accuracy based on network conditions

---

## Usage Examples

```bash
# Quick scan of top 100 ports on localhost
./portscan -hosts 127.0.0.1

# Scan specific ports with service detection
./portscan -hosts 192.168.1.10 -ports 1-1024,8080,8443 -service

# Scan entire subnet with JSON output
./portscan -hosts "192.168.1.0/24" -ports top:200 -json -o results.json

# Include closed ports in results
./portscan -hosts example.com -ports top:100 -all

# High-speed scan with aggressive settings
./portscan -hosts 10.0.0.5 -ports top:1000 -workers 800 -timeout 300ms
```

---

## Technical Highlights

### Concurrency Pattern
The scanner implements a classic producer-consumer pattern with goroutines:
- A producer goroutine feeds port numbers into a jobs channel
- Worker goroutines consume from the jobs channel and write results
- A WaitGroup ensures all workers complete before results are collected

### Banner Sanitization
To handle unprintable characters in service banners:
```go
func sanitizeBanner(s string) string
```
This function filters non-printable ASCII characters and normalizes whitespace, ensuring clean output.

### Automatic Color Detection
The output formatter detects if stdout is a TTY and automatically enables/disables ANSI color codes:
```go
func isTTY(w io.Writer) bool
```
This ensures clean output whether printing to a terminal or redirecting to a file.

---

## Lessons Learned

### 1. **Concurrency Trade-offs**
Finding the right worker pool size is crucial. Too few workers and scans are slow; too many can overwhelm networks or trigger rate limiting. The default of 500 workers with 500ms timeout works well for most networks.

### 2. **Timeout Sensitivity**
Different services respond at different speeds. HTTPS handshakes require more time than simple TCP connections. The scanner uses fractional timeouts for multi-step probing (e.g., TLS then HTTP).

### 3. **Clean Abstractions**
Organizing code into `internal/scanner`, `internal/output`, and `internal/ports` packages kept the codebase maintainable and testable. Each package has a single, clear responsibility.

---

## Future Enhancements

Some ideas for future development:
- UDP port scanning support
- SYN scanning with raw sockets (requires privileges)
- Nmap-style OS fingerprinting
- Plugin system for custom service detection
- Progress indicators for long scans
- Rate limiting controls

---

## Conclusion

Building this port scanner was an excellent learning experience that reinforced my understanding of networking fundamentals, Golang, and practical security reconnaissance techniques. The project demonstrates how relatively simple code can create a powerful, useful, and fast security tool.

The complete source code is available on [GitHub](https://github.com/nicoleman0/portscanner-go).

Whether you're learning Go, exploring network security, or just need a lightweight port scanner, I hope this project proves useful or educational.
