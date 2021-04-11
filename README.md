# Mitra

A tool to generate binary polyglots
(files that are valid with several file formats).

Loosely named after [Μιθραδάτης](https://en.wikipedia.org/wiki/Mithridates_VI_of_Pontus),
a famous polyglot.
Pronounced `mɪtrə`.

[What's new](NEWS.md).

## How to use

`mitra.py file1.png file2.dcm` gives you a working PNG/DICOM polyglot.

Check Corkami [mini](https://github.com/corkami/pocs/tree/master/mini)
or [tiny](https://github.com/corkami/pocs/tree/master/tiny) PoCs for input files.
and the formats [repository](https://github.com/corkami/formats/tree/WIP) for some extra technical info.


# Features

It tries different layouts:
Stacks (appended data), Cavities (blank space), Parasites (comments), Zippers (mutual comments).

It returns the offsets where the payloads 'switch sizes' for multi-ciphertexts.

Ex: `Z(80-162-286)-DICOM^TIFF.be3b767b.dcm.tif` is a DICOM/TIFF zipper
where the payloads switch side at offsets `0x80`, `0x162` and `0x286`.

The `-s` option extracts the 2 payloads separately, mixed with pseudo-random bytes
(it doesn't fix checksums afterwards).

Mitra doesn't generate crypto-polyglots or any polyglots with overlapping bytes,
but it can definitely help to craft them.


# Goals


## Exploration

The goal of this project was to explore polyglots layouts,
formats abuses, and understand which format features enables which kind of abuse.

For example, in a chunk-based format, just find where to `cut` the file,
then `wrap` foreign data in a new chunk and insert the chunk.
So you just need to teach Mitra how to `identify` the type,
where to `cut`, and how to `wrap`.


## Demonstration

You can prevent a software launch if you find a security risk,
but a bad format design is not considered a risk in itself:
it's just the parser's fault, not the designer's !

So if you review a bad file format,
then maybe with Mitra you can quickly generate polyglots with many other formats
and demonstrate the risks.


# Status

```
                          Delayed                 Magic at offset zero,                  No appended
      Any offset  Cavities start                 tolerated appended data                     data    Footer

        Z 7 A R   P I D T   P M   A B B C C E E F F G G I I I I J J N O P L P P R R T W   B J P P W   I X
        i Z r A   D S C A   S P   R M Z A P B L L l I Z C C D L P P E G S N E N I T I A   P a C C A   D Z
        p   j R   F O M R     4     P 2 B I M F V a F   C O 3 D 2 G S G D K   G F F F D   G v A A S   3
                                          O L     c         v A                 F   F       a P P M   v
                                                            2                                   N     1

Zip     . X X X   X X X X   X X   X X X X X X X X X X X X X X X X X X X X X X X X X X X   X X X X X       41
7Z      X . X X   X X X X   X X   X X X X X X X X X X X X X X X X X X X X X X X X X X X   X X X X X       41
Arj     X X . X   X X X X   X X   X X X X X X X X X X X X X X X X X X X X X X X X X X X   X X X X X       41
RAR     X X X .   X X X X   X X   X X X X X X X X X X X X X X X X X X X X X X X X X X X   X X X X X       41

PDF     X X X X   . X X X   X X   X X X X X X X X X X X X X X X X X X X X X X X X X X X   X X X X X       41
ISO     X X X X   X . X X   X X   X X X X X X X X X X X X X X X X X X X X X X X X X X X   X X X X X       41
DCM     X X X X   X X .     X X   X X X X X   X X X X X   X X X X X X X X   X X X X X X   X X X X X       37
TAR     X X X X   X X   .   X X   X     X X     X X   X   X X X X X   X X     X X X X X   X   X X X       30

PS      X X X X   X X X X   .                                                                              8
MP4     X X X X   X X X X     .                                                                            8

AR      X X X X   X X X X         .                                                                        8
BMP     X X X X   X X X             .                                                                      7
BZ2     X X X X   X X X               .                                                                    7
CAB     X X X X   X X X X               .                                                                  8
CPIO    X X X X   X X X X                 .                                                                8
EBML    X X X X   X X                       .                                                              6
ELF     X X X X   X X X                       .                                                            7
FLV     X X X X   X X X X                       .                                                          8
Flac    X X X X   X X X X                         .                                                        8
GIF     X X X X   X X X                             .                                                      7
GZ      X X X X   X X X X                             .                                                    8
ICC     X X X X   X X                                   .                                                  6
ICO     X X X X   X X X X                                 .                                                8
ID3v2   X X X X   X X X X                                   .                                              8
ILDA    X X X X   X X X X                                     .                                            8
JP2     X X X X   X X X X                                       .                                          8
JPG     X X X X   X X X X                                         .                                        8
NES     X X X X   X X X                                             .                                      7
OGG     X X X X   X X X X                                             .                                    8
PSD     X X X X   X X X X                                               .                                  8
LNK     X X X X   X X                                                     .                                6
PE      X X X X   X X X                                                     .                              7
PNG     X X X X   X X X X                                                     .                            8
RIFF    X X X X   X X X X                                                       .                          8
RTF     X X X X   X X X X                                                         .                        8
TIFF    X X X X   X X X X                                                           .                      8
WAD     X X X X   X X X X                                                             .                    8

BPG     X X X X   X X X X                                                                 .                8
Java    X X X X   X X X                                                                     .              7
PCAP    X X X X   X X X X                                                                     .            8
PCAPN   X X X X   X X X X                                                                       .          8
WASM    X X X X   X X X X                                                                         .        8

ID3v1                                                                                                 .    0
XZ                                                                                                      .  0

Formats combinations: 286
```

Notes that some formats are containers and apply to several file types.


## Overlap polyglots

With the `--overlap` option (disabled by default), filetypes starting at the same offsets can be combined.

The overwritten content might be restored via any operation, such as decryption via specific bruteforced parameters (CBC, GCM...), and is stored in the filename between curly brakets. Ex: `O(5-204){424D4E0100}.bmp.jpg`

Since it's pure bruteforcing, it's not practical if the filetype requires too many bytes to be restored.

```
                                                               Variable  Unsupported
                                                                offset     parasite
Minimal start offset
         1 2 4 8     9  16  20  23  28  34  40  64  94  132    12  28   
                      12          26  32  36      68  112 226    16     

         P P J F M T F W G P R I R B C I P C J P E A P I I J    W B O      B E G L N
         S E P l P I L A Z N I D T M P L S A P C L R C C C a    A P G      Z B I N E
             G a 4 F V D   G F 3 F P I D D B 2 A F   A O C v    S G G      2 M F K S
               c   F         F v     O A       P     P     a    M            L
                               2               N
                                               G

1* PS    . M A ? ? ? ? ? ? A ? ? ? ? ? ? ? ? ? ? ? ? ? ? ? ?    ? ? ?      ? ? ? ? ?
2^ PE    M . A A A A A A A A A A A A A A A A A A ! ! ! ! ! !    M M M      ! ! ! ! !
4+ JPG   A A . A A A A A A A A A A A A A A A A A A A A A A A    A A A      A A A A A
.  .
.  .     [the table could go on but would take too long to bruteforce]

           X: automated     ?: likely possible
           M: manual        !: unknown
```

- `*`: hack that relies on line comments with GhostScript - requires the parasite not to contain any new line, **after** encryption.
- `^`: hack relying on overwritting the `Dos Header`, therefore restricting the parasite space to offsets `2`-`60`.
- `+`: Signature, comment declaration and length takes all 2 bytes each, in total:
  - to set an exact length, `6` bytes are required.
  - the length is big endian, so you can round up the length and leave the lowest byte uncontrolled, requiring only `5` bytes.
  - If the entire length is left undefined, and if your payload fits in the resulted encoded length after encryption, then only `4` bytes are required.


# Notes


## File formats

Remarks and recommendations to design file formats

- Enforcing a magic at offset zero should be standard.
- Starting at offset zero and not enforcing a magic at zero is still exploitable (PS, MP4).
- Starting at any offset makes polyglots trivial.
- Enforcing a footer (like `XZ`, `ID3v1`) is a great way to check if a file isn't truncated,
and prevents 'naturally' appended data.
- Most formats have a way to store parasite data, except very simple ones.
- Don't use a standard container in a non standard way.
Use a different magic if you're only going to use similar structures to a limited extend.


## Polyglots detection

Since these polyglots contain the magic signatures of both formats,
`file` may be able to detect them with the `--keep-going` arguments.
(it does not *validate* the formats, but at least gives you some information).

```
$ file --keep-going --raw "C(66)-PNG-DICOM.7e22f58e.dcm.png"
C(66)-PNG-DICOM.7e22f58e.dcm.png: PNG image data, 13 x 7, 1-bit colormap, non-interlaced
- DICOM medical imaging data
- data
```
