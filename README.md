# Manual DNS

This project implements **manual DNS** smart contract, along with scripts needed to register subdomains and change smart contract ownership. 

DNS values are parsed from specially formatted text files which let user store text or binary data. This submission includes scripts for generating public keys and local testing.

Feel free to [get in touch](https://t.me/nikstar). This project is hosted [on Github](https://github.com/nikstar/ton-dns), I will make it public after submission deadline. Check there for updates!

## Creating new contract

1. Compile contract code:
    
    ```bash
    func -PS -o manual-dns.fif stdlib.fc stdlib_ext.fc manual-dns.fc
    ```

    [manual-dns.fc](manual-dns.fc) includes smc code and [stdlib_ext.fc](stdlib_ext.fc) includes some asm extensions.

    Note that `manual-dns.fif` is already checked in this repo, so this step can be skipped.

2. Create new contract:

    ```bash
    fift -s new-manual-dns.fif 0 files/id1
    ```

3. Send Grams to non-bouncable address and upload.

4. Get-method `seqno` should return 1 on success.

## Registering new subdomain

New subdomains can be registered using [register.fif](register.fif):

```bash
fift -s register.fif files/id1.addr out/id1.pk "ru." files/entries.txt 1 files/reg1.boc
```

(Run without options for help.)

In this example DNS values are parsed from following text file:

```
-2 => file out/id1.pub
-1 => address kQCnvV61wU5cLVghtZGlgGs_uS-Kzhgc1wp3K-3pyxH7z6xv
 1 => text Hello darkness, my old friend
 2 => base64 SSd2ZSBjb21lIHRvIHRhbGsgd2l0aCB5b3UgYWdhaW4K
 3 => base64url QmVjYXVzZSBhIHZpc2lvbiBzb2Z0bHkgY3JlZXBpbmc=
 4 => hex 4c65667420697473207365656473207768696c6520492077617320736c656570696e670a
```

As you can see, values can be provided in any desired format: imported from file or provided directly as text or encoded data.

This format can be easily extended for further convenience once more details of DNS are nailed down. For example, aliases for predefined categories can be used: `-2 constant OWNER`, `1 constant CNAME` and so on. See [register.fif](register.fif) for implementation details.

### Unregistering subdomains

Subdomains can be deleted using [unregister.fif](unregister.fif):

```bash
fift -s unregister.fif files/id1.addr out/id1.pk "ru." 2 files/unreg1.boc
```

This is equivalent to passing an empty file to `register.fif`.

## Dnsresolve

Dnsresolve get-method has been implemented according to spec. In this example

- `dnsresolve "ru." 0` returns (3, dictionary from category to dns-value),
- `dnsresolve "ru." 1` returns (3, dns-value),
- `dnsresolve "ru." 42` returns (-1, null),
- `dnsresolve "ru.yandex" 0` returns (3, dns-value for category=-1) (if category=-1 is not set, returns (-1, null)),
- `dnsresolve "org." 0` returns (-1, null).

One thing I was not clear about is handling string with multiple subdomain names, e.g. `ru\0org\0`. I opted to return value for the first subdomain (`ru` in this case) and ignore the rest of the string. If smc is expected to return values for *any* of the names provided, it is unclear what to return as the first value (number of bytes consumed).

## Changing ownership

Owner (i.e. public key stored in smc persistent storage) can be changed using [change-owner.fif](change-owner.fif):

```bash
fift -s change-owner.fif files/id1.addr files/id1.pk files/id2.pub 3 files/change12.boc
```

Note that public key file can be obtained by running `fift -s pub.fif files/id2`. Current owner does not necessarily have access to new private key, so this interface was chosen.
