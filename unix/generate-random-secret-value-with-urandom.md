# Generate Random Secret Value With urandom

The `/dev/urandom` device that is available on Unix systems provides an endless
stream of cryptographically secure pseudorandom bytes. This means that at any
given moment we can read some bytes from that device to get some random output.
The sources of entropy that ensure the randomness are things like interrupt
signals from various pieces of system hardware, unpredictable user input timing,
and other sensors on the device (e.g. value from an accelerometer).

Let's start by reading 32 bytes from `/dev/urandom`:

```bash
❯ head -c 32 /dev/urandom
VY�����ׄ�M�,R�e�!ٸ"���W�%
```

That's a bunch of random bytes and binary nonsense that my terminal can hardly
render.

To turn that into something usable, I need to pipe it through `xxd` which is a
tool for creating a hex dump from a file, or in our case, `stdin`.

The `-p` flag tells `xxd` to output in _plain_ hex dump style. Whereas the `-c 32` ensures that 32 bytes can be printed on a single line (which aligns with the
32 bytes we grab with `head`).

```bash
❯ head -c 32 /dev/urandom | xxd -p -c 32
adf186884769b03fb07b92fd2aeaccad9fadbde97f011cfb24d11e8a8c72bcaa
```

Notice what happens if we are missing either of those flags:

```bash
❯ head -c 32 /dev/urandom | xxd
00000000: bfca e986 a309 4ec8 ea01 d36b 2b4a f9fb  ......N....k+J..
00000010: 5f99 bb87 95fe f7fb 6a89 cc62 a64f 1c7d  _.......j..b.O.}

❯ head -c 32 /dev/urandom | xxd -p
60d48b1fd1d85e3bf74b62c840aebf6db8a263670f94ceaf0ba63b380e56
94fb
```

I use this or [similar techniques](generate-a-random-secret-value-with-openssl.md) to generate a random
secret value for web app development when I need two systems to have a shared
secret they can use to verify requests.
