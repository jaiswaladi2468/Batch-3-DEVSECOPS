# Jenkins Shared Library

A Jenkins shared library is a powerful feature in Jenkins that allows you to define and reuse pipeline code across multiple projects. It promotes code reuse, maintainability, and consistency in your Jenkins pipeline scripts. Shared libraries are typically stored in version control systems like Git, making it easy to manage and version the code.

Here's a detailed overview of Jenkins shared libraries:

### 1. **What is a Shared Library?**
   - A shared library in Jenkins is a collection of reusable pipeline code written in Groovy.
   - It allows you to define functions, variables, and classes that can be used across multiple Jenkins pipelines.

### 2. **Setting Up a Shared Library:**
    Refer https://github.com/jaiswaladi246/sharedlib.git
   - Create a new Git repository to store your shared library code.
   - The repository structure typically includes directories like `src` for Groovy source code and `vars` for global variables.
   - The `src` directory might contain classes and utility functions.
   - The `vars` directory contains global variables that can be directly accessed in pipelines.

### 3. **Defining Functions and Classes:**
   - Write Groovy code in your shared library to define reusable functions and classes.
   - Functions in the `vars` directory become global variables in Jenkins pipelines.

### 4. **Loading the Shared Library:**
   - In a Jenkins pipeline, you can load the shared library using the `@Library` annotation.
   - Example:
     ```groovy
     @Library('my-shared-library') _
     ```

### 5. **Accessing Shared Library Functions:**
   - Once the library is loaded, you can call its functions in your Jenkins pipeline as if they were native pipeline steps.

### 6. **Pipeline Example with Shared Library:**
   ```groovy
   @Library('my-shared-library') _

   pipeline {
       agent any
       
       stages {
           stage('Build') {
               steps {
                   // Using shared library function
                   mySharedLibraryFunction()
               }
           }
           stage('Test') {
               steps {
                   // Using shared library class
                   script {
                       def myObject = new com.example.MyClass()
                       myObject.myMethod()
                   }
               }
           }
       }
   }
   ```

### 7. **Versioning and Updates:**
   - Manage versions of your shared library in your version control system (e.g., Git tags).
   - Jenkins allows you to specify the version of the library to use in your pipeline.

### 8. **Global Variables:**
   - Variables defined in the `vars` directory become global variables in the pipeline.
   - Example: If you have a file named `myGlobalVar.groovy` in `vars`, you can access the variable as `myGlobalVar` in your pipeline.

### 9. **Security Considerations:**
   - Shared libraries can be configured to run in a sandbox or trusted mode, depending on your security requirements.
   - Carefully control access to your shared library and limit who can modify it.

### 10. **Documentation:**
   - Include documentation for your shared library, explaining how to use each function and any required parameters.

### 11. **Testing:**
   - Implement unit testing for your shared library to ensure reliability and maintainability.

By using shared libraries, Jenkins users can build more maintainable and scalable CI/CD pipelines. The shared library code can be maintained independently, allowing updates to be easily propagated to all pipelines that use it.
