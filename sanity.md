## Rev/Sanity

Note: The function names here are already renamed so you will have to find them on your own.

You can find the main function by looking at the `start` function. The main function is the first function run there.
```
void __noreturn start()
{
  int Type; // [esp+0h] [ebp-1Ch]
  const char **v1; // [esp+4h] [ebp-18h]
  const char **v2; // [esp+8h] [ebp-14h]

  _set_app_type(_crt_console_app);
  main(Type, v1, v2);
}
```

Inside the main function:
```
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  UINT v3; // ebx

  if ( TlsCallback_0 )
    TlsCallback_0(0, 2, 0);
  SetUnhandledExceptionFilter(TopLevelExceptionFilter);
  sub_401A30();
  sub_402240(dword_405A04);
  sub_401690();
  if ( Mode )
  {
    dword_405A08 = Mode;
    setmode(iob[0]._file, Mode);
    setmode(iob[1]._file, Mode);
    setmode(iob[2]._file, Mode);
  }
  *_p__fmode() = dword_405A08;
  sub_402040();
  sub_401BC0();
  _p__environ();
  v3 = check(dword_408004);
  cexit();
  ExitProcess(v3);
}
```

So this is the last function in main, the check function. You can tell the other ones aren't important because they don't modify a variable, modify a file, or return anything. That leaves this function:
```
v3 = check(dword_408004);
```
to be our checking function.

```
int __cdecl check(char a1)
{
  unsigned int i; // esi
  int j; // edi
  char v4[273]; // [esp+17h] [ebp-121h] BYREF
  char *v5; // [esp+128h] [ebp-10h]

  v5 = &a1;
  sub_401BC0();
  strcpy(v4, "HeHeBoii");
  printf("Enter Flag : ");
  scanf("%256s", &v4[9]);
  for ( i = 0; strlen(aBsnoidaZsbrbm9) > i; ++i )
  {
    printf("\r%*s\r%s", 9, (const char *)&unk_406081, "Checking");
    fflush(&iob[1]);
    if ( ((unsigned __int8)v4[i + 9] ^ (unsigned __int8)aBsnoidaZsbrbm9[i]) != v4[(int)i % 8] )
    {
      puts("\nWrong");
      return 0;
    }
    for ( j = 0; j <= 2; ++j )
    {
      Sleep(0x64u);
      fputc(46, &iob[1]);
      fflush(&iob[1]);
    }
  }
  puts("\nCorrect");
  return 0;
```

It stores our input in `v4[9]`, has the string `HeHeBoii` in v4, and uses the string `aBsnoidaZsbrbm9` with value `BSNoida{ZSBrbm93IHRoZSBnYW1lIGFu}`. Our main checking function is here:
```
if ( ((unsigned __int8)v4[i + 9] ^ (unsigned __int8)aBsnoidaZsbrbm9[i]) != v4[(int)i % 8] )
```

### Xor 1:

It takes our input and xors it with the string and checks if it's equal to the string. So basically:
```
B ^ input[0] == H
S ^ input[1] == e
N ^ input[2] == H
o ^ input[3] == e
i ^ input[4] == B
d ^ input[5] == o
a ^ input[6] == i
{ ^ input[7] == i
Z ^ input[8] == H
S ^ input[9] == e
B ^ input[10] == H
r ^ input[11] == e
b ^ input[12] == B
m ^ input[13] == o
9 ^ input[14] == i
3 ^ input[15] == i
I ^ input[16] == H
H ^ input[17] == e
R ^ input[18] == H
o ^ input[19] == e
Z ^ input[20] == B
S ^ input[21] == o
B ^ input[22] == i
n ^ input[23] == i
Y ^ input[24] == H
W ^ input[25] == e
1 ^ input[26] == H
l ^ input[27] == e
I ^ input[28] == B
G ^ input[29] == o
F ^ input[30] == i
u ^ input[31] == i
} ^ input[32] == H
```

With input satisfying all these conditions, however, running this gives us some weird feedback something like this:
```

6♠
↕↕6
↨ ☻PZ☺-→
```

This is not a flag. So after a few more minutes of head scratching and digging around, I find this function that modifies the string we originally had by checking the xrefs.

### Xor 2:

```
unsigned int mod_check()
{
  unsigned int i; // edx
  unsigned int result; // eax
  char v2[27]; // [esp+Fh] [ebp-25h]
  char v3[6]; // [esp+2Ah] [ebp-Ah] BYREF

  v2[0] = 72;
  v2[1] = 101;
  v2[2] = 72;
  v2[3] = 101;
  v2[4] = 66;
  v2[5] = 111;
  v2[6] = 105;
  v2[7] = 105;
  v2[8] = 87;
  v2[9] = 76;
  v2[10] = 112;
  v2[11] = 109;
  v2[12] = 90;
  v2[13] = 120;
  v2[14] = 42;
  v2[15] = 32;
  v2[16] = 123;
  v2[17] = 87;
  v2[18] = 96;
  v2[19] = 112;
  v2[20] = 98;
  v2[21] = 70;
  v2[22] = 81;
  v2[23] = 125;
  v2[24] = 107;
  v2[25] = 72;
  v2[26] = 3;
  qmemcpy(v3, "VMDN{H", sizeof(v3));
  for ( i = 0; ; ++i )
  {
    result = strlen(aBsnoidaZsbrbm9);
    if ( result <= i )
      break;
    aBsnoidaZsbrbm9[i] ^= v2[i];
  }
  return result;
```
This looks interesting. The first 8 characters spell out `HeHeBoii` in ascii. In the loop, we see that we're just taking the all the characters in the string `aBsnoidaZsbrbm9` and xoring it with this new set of numbers and putting it back into its place. Since those few letters have the same values as `HeHeBoii` we can safely assume this is where we need to go since xoring something by the same thing twice returns the original value, and the `BSNoida{` flag wrapper should stay consistent, meaning that this is the right path. The `qmemcpy` doesn't actually do anything important so we can just ignore that.

Since we understand what's happening now, it's time to reverse:
```
check = [ord(c) for c in 'BSNoida{ZSBrbm93IHRoZSBnYW1lIGFu}']
e = [int(x,16) for x in '48 65 48 65 42 6F 69 69 57 4C 70 6D 5A 78 2A 20 7B 57 60 70 62 46 51 7D 6B 48 3 56 4D 44 4E 7B 48'.split(' ')]
v4 = [ord(c) for c in 'HeHeBoii']
flag = []
for c in range(len(e)):
    check[c] ^= e[c]
```
(Those hex values are simply the `v2` values from above but in hex)
Initialize the data and reverse Xor 2.

```
for x in range(len(check)):
    flag.append(check[x]^v4[x%8])
```
Reverse the Xor 1.

```
print(''.join([chr(x) for x in flag]))
```
Print our results.
`BSNoida{Ezzzzzzzzzzzzzzzzzz_Flag}`

## Flag:
`BSNoida{Ezzzzzzzzzzzzzzzzzz_Flag}`