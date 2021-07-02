1. Найдите полный хеш и комментарий коммита, хеш которого начинается на `aefea`.
```
$ git show -q --pretty=tformat:'%H %s' aefea
aefead2207ef7e2aa5dc81a34aedf0cad4c32545 Update CHANGELOG.md
```

2. Какому тегу соответствует коммит `85024d3`?
v0.12.23
```
$ git show -q --pretty=tformat:'%H %s %d' 85024d3
85024d3100126de36331c6982bfaac02cdab9e76 v0.12.23  (tag: v0.12.23)
```

3. Сколько родителей у коммита `b8d720`? Напишите их хеши.
Два.
```
$ git rev-parse b8d720^@
56cd7859e05c36c06b56d013b55a252d0bb7e158
9ea88f22fc6269854151c571162c5bcf958bee2b
```

4. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами  v0.12.23 и v0.12.24.
```
$ git show -q --pretty=tformat:'%H %s' v0.12.23..v0.12.24
33ff1c03bb960b332be3af2e333462dde88b279e v0.12.24
b14b74c4939dcab573326f4e3ee2a62e23e12f89 [Website] vmc provider links
3f235065b9347a758efadc92295b540ee0a5e26e Update CHANGELOG.md
6ae64e247b332925b872447e9ce869657281c2bf registry: Fix panic when server is unreachable
5c619ca1baf2e21a155fcdb4c264cc9e24a2a353 website: Remove links to the getting started guide's old location
06275647e2b53d97d4f0a19a0fec11f6d69820b5 Update CHANGELOG.md
d5f9411f5108260320064349b757f55c09bc4b80 command: Fix bug when using terraform login on Windows
4b6d06cc5dcb78af637bbb19c198faff37a066ed Update CHANGELOG.md
dd01a35078f040ca984cdd349f18d0b67e486c35 Update CHANGELOG.md
225466bc3e5f35baa5d07197bbc079345b77525e Cleanup after v0.12.23 release
```

5. Найдите коммит в котором была создана функция `func providerSource`, ее определение в коде выглядит 
так `func providerSource(...)` (вместо троеточего перечислены аргументы).
```
$ git log -S 'func providerSource(' --oneline
8c928e835 main: Consult local directories as potential mirrors of providers
```

6. Найдите все коммиты в которых была изменена функция `globalPluginDirs`.
```
$ git log --oneline -L :globalPluginDirs:plugins.go
78b122055 Remove config.go and update things using its aliases

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,14 +18,14 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
-       dir, err := ConfigDir()
+       dir, err := cliconfig.ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
                ret = append(ret, filepath.Join(dir, "plugins"))
                ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }
 
        return ret
 }
52dbf9483 keep .terraform.d/plugins for discovery

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,13 +16,14 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
+               ret = append(ret, filepath.Join(dir, "plugins"))
                ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }
 
        return ret
 }
41ab0aef7 Add missing OS_ARCH dir to global plugin paths

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -14,12 +16,13 @@
 func globalPluginDirs() []string {
        var ret []string
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
-               ret = append(ret, filepath.Join(dir, "plugins"))
+               machineDir := fmt.Sprintf("%s_%s", runtime.GOOS, runtime.GOARCH)
+               ret = append(ret, filepath.Join(dir, "plugins", machineDir))
        }
 
        return ret
 }
66ebff90c move some more plugin search path logic to command

diff --git a/plugins.go b/plugins.go
--- a/plugins.go
+++ b/plugins.go
@@ -16,22 +14,12 @@
 func globalPluginDirs() []string {
        var ret []string
-
-       // Look in the same directory as the Terraform executable.
-       // If found, this replaces what we found in the config path.
-       exePath, err := osext.Executable()
-       if err != nil {
-               log.Printf("[ERROR] Error discovering exe directory: %s", err)
-       } else {
-               ret = append(ret, filepath.Dir(exePath))
-       }
-
        // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
        dir, err := ConfigDir()
        if err != nil {
                log.Printf("[ERROR] Error finding global config directory: %s", err)
        } else {
                ret = append(ret, filepath.Join(dir, "plugins"))
        }
 
        return ret
 }
8364383c3 Push plugin discovery down into command package

diff --git a/plugins.go b/plugins.go
--- /dev/null
+++ b/plugins.go
@@ -0,0 +16,22 @@
+func globalPluginDirs() []string {
+       var ret []string
+
+       // Look in the same directory as the Terraform executable.
+       // If found, this replaces what we found in the config path.
+       exePath, err := osext.Executable()
+       if err != nil {
+               log.Printf("[ERROR] Error discovering exe directory: %s", err)
+       } else {
+               ret = append(ret, filepath.Dir(exePath))
+       }
+
+       // Look in ~/.terraform.d/plugins/ , or its equivalent on non-UNIX
+       dir, err := ConfigDir()
+       if err != nil {
+               log.Printf("[ERROR] Error finding global config directory: %s", err)
+       } else {
+               ret = append(ret, filepath.Join(dir, "plugins"))
+       }
+
+       return ret
+}
```

7. Кто автор функции `synchronizedWriters`? 
Martin Atkins
```
$ git log -S 'func synchronizedWriters(' -p --pretty=tformat:'%an'
James Bardin

diff --git a/synchronized_writers.go b/synchronized_writers.go
deleted file mode 100644
index 2533d1316..000000000
--- a/synchronized_writers.go
+++ /dev/null
@@ -1,31 +0,0 @@
-package main
-
-import (
-       "io"
-       "sync"
-)
-
-type synchronizedWriter struct {
-       io.Writer
-       mutex *sync.Mutex
-}
-
-// synchronizedWriters takes a set of writers and returns wrappers that ensure
-// that only one write can be outstanding at a time across the whole set.
-func synchronizedWriters(targets ...io.Writer) []io.Writer {
-       mutex := &sync.Mutex{}
-       ret := make([]io.Writer, len(targets))
-       for i, target := range targets {
-               ret[i] = &synchronizedWriter{
-                       Writer: target,
-                       mutex:  mutex,
-               }
-       }
-       return ret
-}
-
-func (w *synchronizedWriter) Write(p []byte) (int, error) {
-       w.mutex.Lock()
-       defer w.mutex.Unlock()
-       return w.Writer.Write(p)
-}
Martin Atkins

diff --git a/synchronized_writers.go b/synchronized_writers.go
new file mode 100644
index 000000000..2533d1316
--- /dev/null
+++ b/synchronized_writers.go
@@ -0,0 +1,31 @@
+package main
+
+import (
+       "io"
+       "sync"
+)
+
+type synchronizedWriter struct {
+       io.Writer
+       mutex *sync.Mutex
+}
+
+// synchronizedWriters takes a set of writers and returns wrappers that ensure
+// that only one write can be outstanding at a time across the whole set.
+func synchronizedWriters(targets ...io.Writer) []io.Writer {
+       mutex := &sync.Mutex{}
+       ret := make([]io.Writer, len(targets))
+       for i, target := range targets {
+               ret[i] = &synchronizedWriter{
+                       Writer: target,
+                       mutex:  mutex,
+               }
+       }
+       return ret
+}
+
+func (w *synchronizedWriter) Write(p []byte) (int, error) {
+       w.mutex.Lock()
+       defer w.mutex.Unlock()
+       return w.Writer.Write(p)
+}
```

