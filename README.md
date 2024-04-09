# CTF APK Manipulation Cheatsheet

This cheatsheet covers essential commands for decompiling, modifying, recompiling, and signing Android APK files, a common task in CTF challenges involving Android applications.

## Prerequisites

Ensure you have `apktool` and `Java Development Kit (JDK)` installed on your system. `apktool` is used for decompiling and recompiling APK files, and `JDK` provides `keytool` and `apksigner` for key management and APK signing.

## Commands

### Decompiling APK

To decompile an APK file and extract its contents for analysis and modification:

```
apktool d practice-CTF.apk
```

- **d**: stands for decode; it tells apktool to decompile the APK.
- **practice-CTF.apk**: the APK file you want to decompile.

### Recompiling APK

After making your modifications, recompile the APK with:

```
apktool b practice-CTF -o practice-CTF-recompiled2.apk
```

- **b**: stands for build; it tells apktool to recompile the APK.
- **practice-CTF**: the directory of the decompiled APK files.
- **-o**: specifies the output file name.
- **practice-CTF-recompiled2.apk**: the name of the recompiled APK file.

### Signing APK

Sign the recompiled APK with a keystore for installation on an Android device:

```
apksigner sign --ks key.jks practice-CTF-recompiled2.apk
```

- **sign**: command to sign the APK.
- **--ks**: specifies the keystore file.
- **key.jks**: the keystore file used for signing the APK.
- **practice-CTF-recompiled2.apk**: the APK file to be signed.

### Generating Keystore

If you need a new keystore for signing your APKs:

```
keytool -genkey -v -keystore key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-alias
```

- **-genkey**: command to generate a new key.
- **-v**: enables verbose output.
- **-keystore**: specifies the keystore file to create or use.
- **key.jks**: the name of the keystore file.
- **-keyalg**: the algorithm used for the key.
- **RSA**: specifies RSA algorithm.
- **-keysize**: size of the key in bits.
- **2048**: the bit length of the key.
- **-validity**: the validity period of the key in days.
- **10000**: number of days the key is valid for.
- **-alias**: the alias name for the key.
- **my-alias**: your chosen alias for the key.

## Notes

- Always backup original APKs before modification.
- Signing with a keystore is essential for installing modified APKs on Android devices.
- Ensure the `key.jks` and the alias used for signing are consistent across your APKs for simplicity.

## Adding Log Statements in Smali

To add logging in Smali bytecode, you need to use the `Log.d` method. Below are examples of how to log different types of variables: `String`, `int`, and `long`.

### Logging a String Variable

To log a `String` variable:

```smali
const-string v0, "My Logging Message"

# Assume v1 holds the string variable you want to log
invoke-static {v0, v1}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I
```

### Logging an int Variable

To log an `int` variable, you first need to convert the `int` to a `String`:

```smali
const-string v0, "My Logging Message"

# Assume v1 holds the int variable you want to log
invoke-static {v1}, Ljava/lang/String;->valueOf(I)Ljava/lang/String;

move-result-object v1

invoke-static {v0, v1}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I
```

### Logging a long Variable

To log a `long` variable, similar to logging an `int`, you need to convert the `long` to a `String`:

```smali
const-string v0, "My Logging Message"

# Assume v1 and v2 together hold the long variable you want to log
invoke-static {v1, v2}, Ljava/lang/String;->valueOf(J)Ljava/lang/String;

move-result-object v1

invoke-static {v0, v1}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I
```

Note: In Smali, `long` and `double` types occupy two registers (e.g., `v1` and `v2`) because they are 64 bits long.

### General Notes for Logging

- Ensure you have the appropriate permissions and configurations in your app to view log messages.
- The `invoke-static` instruction is used for calling static methods, such as `Log.d`.
- The `move-result-object` instruction moves the result of the last method call into a register.
- Adjust the register numbers (`v0`, `v1`, `v2`, etc.) based on your method's register allocation.

Incorporate these snippets into your Smali files as needed to add logging for different types of variables. Always double-check your register allocations to avoid conflicts and ensure accurate logging.

### What are `.locals`?

In Smali, each method has a `.locals` directive that specifies the number of local variable slots the method uses. Local variables include method parameters, temporary variables used within the method, and any variables needed for method calls within it.

### How `.locals` Works

- **Slot Allocation**: Each variable type, except for `long` and `double`, occupies one slot. `Long` and `double` types require two slots because they are 64-bit data types.
- **Method Arguments**: The count includes the method's arguments. For instance, in instance methods (non-static), `this` is implicitly passed as the first argument and counts towards the `.locals` total.

### Example 1: Basic Method

Let's consider a simple method that takes no arguments and uses one local variable:

```smali
.method public doSomething()V
    .locals 1

    const/4 v0, 0x5
    # v0 is a local variable set to 5

    return-void
.end method
```

- **`.locals 1`** indicates there is 1 local variable (`v0`).

### Example 2: Adding a Local Variable

If we want to add a new `int` variable:

```smali
.method public doSomething()V
    .locals 2  # Increased to 2 because we're adding another variable

    const/4 v0, 0x5
    const/4 v1, 0x6  # Added a new local variable, v1
    # Now we have two local variables: v0 and v1

    return-void
.end method
```

- We incremented `.locals` from 1 to 2 because we added a new variable (`v1`).

### Example 3: Working with Method Parameters

Consider a method with parameters. Parameters are counted in the `.locals` directive:

```smali
.method public addTwoNumbers(II)I
    .locals 1  # Only need one additional local variable for the result

    add-int v0, p1, p2  # p1 and p2 are method parameters, v0 is for the result

    return v0
.end method
```

- **Parameters `p1` and `p2`** are the two integers to be added. They don't need to be explicitly declared in `.locals` because `.locals` only counts additional local variables beyond the parameters.
- We have one local variable (`v0`) for storing the result, hence `.locals 1`.

### Example 4: Modifying a Method to Add Logging

Suppose we modify a method to include logging, which requires additional string variables:

```smali
.method public logResult()V
    .locals 3  # Adjusted for added logging variables

    const/4 v0, 0x7

    # Assume we're logging the result
    const-string v1, "Result"
    const-string v2, "The result is 7"

    invoke-static {v1, v2}, Landroid/util/Log;->d(Ljava/lang/String;Ljava/lang/String;)I

    return-void
.end method
```

- **`.locals 3`**: We need two additional variables for the log tag and message (`v1` and `v2`), besides the original variable (`v0`). Thus, we update `.locals` to 3.

### Best Practices

- **Accuracy**: Ensure the `.locals` count accurately reflects the total number of variables used in the method, including parameters and any added or removed variables.
- **Optimization**: While you might be tempted to allocate more slots "just in case," it's best practice to use only what you need for clarity and efficiency.
- **Testing**: After modifying Smali code, especially changes to `.locals`, test the application thoroughly to ensure it behaves as expected without crashes or errors.