# ESP Development Environment -- Add Self Directory

## There are two ways to add our self-directory

### A .Current project way

#### 1. Create a directory named `components` under current project

#### 2. Create our self directory, just like `test_print` under `components`  directory we created before;

#### 3. Create `.mk`  file named `component.mk` under ``test_print`` directory we created before, edit this `.mk` file, input:

```
COMPONENT_ADD_INCLUDEDIRS := .
```

#### 4. Create `config`  file named `Kconfig` under ``test_print`` directory we created before, edit this `config` file, input:

```
menu "Test PRINT"

config TEST_PRINT_ENABLE
bool "Enable Test Print"
default "y"
endmenu

```

#### 5. Add our self `.c` and `.h` files under `test_print` directory

#### 6. Run `make menuconfig`, in `Composite config` option we will find `Test PRINT` and it's chose default.



### B. Common project way

#### It is differently from way A that we just need to create our self directory, just like `test_print` under the `esp_idf/components` directory



