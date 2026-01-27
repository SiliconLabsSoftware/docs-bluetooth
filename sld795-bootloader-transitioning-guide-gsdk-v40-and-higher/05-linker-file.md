# Linker File

The linker files for GCC and IAR are now generated during the project generation step. The linker files for the bootloader are generated from template files using the Jinja template engine. The template files are included with theGecko SDK Suite (GSDK), which is installed through Simplicity Studio. The jinja template files are located at **\<path-to-simplicity-studio-installation\>\|v5\|developer\|sdks\|gecko\_sdk\_suite\|\<gecko\_sdk\_version\>\|platform\|common\|toolchain\|**.This path contains folders for both **gcc** and **iar**.
