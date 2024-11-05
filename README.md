# Hashcat_Intro
Creating hash MD5SUM and cracking them with Hashcat

## Hashcat Lesson

1. Create a hashed password using md5sum command. You can do it two ways:

    a. The first way is to type the md5sum command and start a shell. The md5sum shell will allow you to enter the cleartext password without recording to shell history.
    ```bash
    md5sum
    ```
    - Type `md5sum` to start the md5sum shell.
    - Type the text string you wish to hash eg: Password123
    - Press ENTER to get a new line
    - Use the combination key `CTRL+D` to exit the shell and execute the hashing program. You should have something similar to the example output below:
    ```bash
    md5sum
    Password123
    a907ac8f85bbece3069a52a39947b287  -
    ```
    - Copy the hash (minus the space at the end) and paste it into a file then save it.

    b. The second method is to use a shell command to generate the hash and save to a file but cut out any remaining characters after the last contiguous set of characters. This command will be recorded to history exposing the cleartext password.
    - Example: 

    ```bash
    printf '%s' 'password' | md5sum | cut -d ' ' -f 1 > hashed-password.txt
    ```

    i. **`printf '%s' 'password'`**:
    - This command prints the string `'password'` without a newline character. The `%s` format specifier tells `printf` to treat the argument as a string.

    ii. **`|` (pipe)**:
    - The pipe operator takes the output of the `printf` command and passes it as input to the next command, `md5sum`.

    iii. **`md5sum`**:
    - This command computes the MD5 hash of the input it receives. In this case, it hashes the string `'password'`. The output is a string of 32 hexadecimal characters followed by a space and a hyphen (indicating the input was from standard input).

    iv. **`|` (pipe)**:
    - Again, the pipe operator takes the output of the `md5sum` command and passes it as input to the next command, `cut`.

    v. **`cut -d ' ' -f 1`**:
    - The `cut` command is used to extract sections from each line of input. Here, `-d ' '` specifies that the delimiter is a space, and `-f 1` tells `cut` to select the first field (the MD5 hash).

    vi. **`> hashed-password.txt`**:
    - The redirection operator `>` takes the final output (the MD5 hash) and writes it to a file named `hashed-password.txt`.

    This command sequence takes the string `'password'`, computes its MD5 hash, extracts the hash value, and saves it to `hashed-password.txt`.

    c. You can cat the file to see the hash that was created.

2. Use hashcat to attempt to find the cleartext equivalent.
Usually we would be unsure about the password length, so we would use a mask increment attack in Hashcat, which allows us to start with shorter lengths and gradually increase to a maximum limit that we specify. Here’s how we can do that:

```bash
hashcat -m 0 -a 3 hashed-password.txt ?a?a?a?a?a?a?a?a --increment --increment-min=1 --increment-max=8
```

Here's a breakdown of the command:
- `-m 0`: Specifies the hash type as MD5.
- `-a 3`: Specifies a brute force attack mode.
- `hashed-password.txt`: The file containing the MD5 hash.
- `?a?a?a?a?a?a?a?a`: The mask for the brute force attack, where `?a` represents all possible characters.
- `--increment`: Enables incremental mode.
- `--increment-min=1`: Starts with a minimum length of 1 character.
- `--increment-max=8`: Sets the maximum length to 8 characters.

This command will start with a 1-character password and incrementally increase the length up to 8 characters. You can adjust the `--increment-max` value if you suspect the password might be longer.

To specify your own custom mask rule in hashcat using the character set `ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789` (no special characters), you can define a custom character set and then use it in your mask. Here's how you can do it:

1. **Define the custom character set**:
   Use the `-1` option to define your custom character set.

2. **Use the custom character set in your mask**:
   Replace the mask symbols with your custom character set.

Here's an example command:

```bash
hashcat -a 3 -m <hash_type> <hash_file> -1 ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789 ?1?1?1?1?1?1?1?1?1?1?1?1?1?1?1?1
```

In this command:
- `-1 ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789`: Defines the custom character set as `?1`.
- `?1?1?1?1?1?1?1?1?1?1?1?1?1?1?1?1`: Uses the custom character set for all 16 positions in the mask.

This setup will attempt to crack passwords that are 16 characters long, using any combination of uppercase letters, lowercase letters, and digits.



NOTE: Hashcat keeps track of previously cracked hashes using a potency file (.potfile). The name comes from the idea that the file contains “potent” or effective results—specifically, the hashes that have been successfully cracked along with their corresponding plaintext passwords. Its an ascii text file so you can cat it to see the contents if you wish

In Kali Linux 6 the location of hashcat's potfile is as follows:

`~/.local/share/hashcat/hashcat.potfile`


So, if you already cracked it hashcat will remember it and you can just show the cleartext equivalent with the following command:

```bash
hashcat -m 0 --show hashed-password.txt
```
