# How to Set or Change the Default Java Version on macOS Using SDKMAN!

### Prerequisites:

- **SDKMAN!** installed. Install SDKMAN! with:
  ```bash
  curl -s "https://get.sdkman.io" | bash
  source "$HOME/.sdkman/bin/sdkman-init.sh"
  ```

# Steps:

### 1. List Available Java Versions:

```bash
sdk list java
```

### 2. Install a Specific Java Version:

    ```bash
    sdk install java <version_identifier>
    ```

### 3. Set a Java Version as Default:

    ```bash
    sdk install java <version_identifier>
    ```

### 4. Switch Temporarily (Current Session):

    ```bash
    sdk use java <version_identifier>
    ```

### 5. Check Active Java Version:

    ```bash
    java -version
    ```

### Resources

1. [SDKMAN official site](https://sdkman.io/install)
2. [Stackoverflow](https://stackoverflow.com/questions/21964709/how-to-set-or-change-the-default-java-jdk-version-on-macos)
