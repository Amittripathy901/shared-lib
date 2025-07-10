## 🚀 What Are Shared Libraries?

A **Shared Library** is a reusable set of **Groovy scripts, classes, and resources** stored in an SCM (like Git). It enables teams to define and reuse common pipeline logic—keeping Jenkinsfiles **DRY**, consistent, and maintainable ([Jenkins][1], [DEV Community][2]).

### Key Benefits:

* **Code Reusability** and consistency across pipelines ([DevOpsCube][3]).
* **Centralized maintenance**: updates in one place affect all pipelines .
* Better **collaboration**, testing, and modularity ([preethi-devops.com][4]).

---

## 🗂️ Directory Structure

According to official docs ([hrmpw.github.io][5]), a Shared Library repo looks like this:

```
(root)
├── src/
|    └── org/foo/Bar.groovy        # Helper classes
├── vars/
|    ├── foo.groovy                # Defines global step `foo`
|    └── foo.txt                   # Optional inline documentation
└── resources/
     └── org/foo/bar.json         # Static resources fetched via libraryResource
```

* **src/**: Groovy classes under typical Java-style package layout.
* **vars/**: Custom pipeline steps; each `.groovy` file defines a global variable/step with a `def call()` signature.
* **resources/**: Non-code assets, such as configs or templates.

---

## ⚙️ Configuring in Jenkins

1. **Add global library** via **Manage Jenkins → Configure System → Global Pipeline Libraries** ([Jenkins][1], [virendra.dev][6]).
2. Provide:

   * **Name**: Identifier used in pipelines (e.g., `myLib`)
   * **SCM settings**: Git repo, default version (branch or tag)
   * **Load behavior**: Implicit loading or explicit needing in Jenkinsfile
3. Optionally configure **Folder-level** libraries to scope per folder ([hrmpw.github.io][5], [virendra.dev][6]).

---

## 📥 Usage in Pipelines

### Declarative:

```groovy
@Library('myLib') _
pipeline {
  agent any
  stages {
    stage('Test') {
      steps {
        foo('hello')                      // `vars/foo.groovy`
        script {
          import org.foo.Bar
          Bar.doSomething()
        }
      }
    }
  }
}
```

### Scripted:

```groovy
@Library('myLib@main') _
foo('world')                          // Calls foo.groovy call()
```

Or use the newer syntax:

```groovy
@Library('myLib@develop') _
import org.foo.Bar
```

For declarative:

```groovy
libraries { lib('myLib') }
```

✅ **Important**: The underscore (`_`) after `@Library(...)` is required if the next line isn’t an `import` ([Cloud Infrastructure Services][7], [tutorialworks.com][8]).

---

## 🛠️ Defining Steps & Helpers

* In **vars/foo.groovy**:

  ```groovy
  def call(String msg = 'hello') {
    echo "foo says: ${msg}"
  }
  ```

* In **src/org/foo/Bar.groovy**:

  ```groovy
  package org.foo
  class Bar {
    static void doSomething() {
      println 'Bar is working'
    }
  }
  ```

* In **resources/org/foo/bar.json**, include static data, retrievable via:

  ```groovy
  def json = libraryResource('org/foo/bar.json')
  ```

---

## 🔧 Best Practices & Tips

* **Version libraries via Git** to manage releases ([virendra.dev][6], [preethi-devops.com][4]).
* **Document steps** using `.txt` files in `vars/`.
* **Sandbox untrusted libraries** via folder-level config ([CloudBees][9], [hrmpw.github.io][5]).
* **Unit test library code** (especially in `src/`), as it runs outside Jenkins .
* **Manage credentials securely**—don’t hardcode them in libraries ([Sling Academy][10]).

---

## 🧪 Example: Code Checkout Step

From Altimetrik's blog:

In `vars/checkoutCode.groovy`:

```groovy
def call(String repoUrl, String branch) {
  def wd = env.WORKSPACE
  sh "git clone ${repoUrl} ${wd}"
  sh "git checkout ${branch}"
  return wd
}
```

In your Jenkinsfile:

```groovy
@Library('myLib') _
pipeline {
  agent any
  stages {
    stage('Checkout') {
      steps {
        script {
          checkoutCode('https://...', 'main')
        }
      }
    }
  }
}
```

✅ A clean reusable checkout function used across pipelines ([altimetrik.com][11], [kiranpawar.hashnode.dev][12]).

---

## ✅ Summary

* **Shared Libraries** centralize reusable pipeline logic.
* Standard structure: `src/`, `vars/`, `resources/`.
* Load via `@Library` or `libraries{}`, then use steps or classes.
* Ideal for enforcing best practices, reducing redundancy, and scaling Jenkins pipelines.

