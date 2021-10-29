# Mutation Testing Lab

## Prerequisites

1. Install Java 8. Note that you will not need to install Maven because you can run Maven wrapper with `./mvnw` (or `./mvnw.cmd` on Windows).
    * _Optional_: Install Java 8 with a version manager tool like [SDKMAN!](https://sdkman.io/install)
    * _Alternative_: Use [jenv](https://www.jenv.be/) on MacOS with [Homebrew](https://brew.sh/)
      ```bash
      brew install java8
      sudo ln -sfn /usr/local/opt/openjdk@8/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-8.jdk
      jenv add /Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/
      jenv shell 8
      ```
2. Fork this repository and clone it
    ```bash
   git clone https://github.com/heidarianw/spring-petclinic
   ```
3. _Optional_: If you have SDKMAN! installed, you can run `sdk env` at the root of the project to configure the environment

Note: This readme is text only. These instructions can be found in the presentation deck as well with added visuals.

## Running

To verify you have set up the application correctly, run the Spring Boot console application.
At the root of the project, run
```bash
./mvnw spring-boot:run
```

## Running Tests

1. Run the unit tests for the project and verify that all tests pass
   ```bash
   ./mvnw clean test
   ```
   You should see the following output in the console
   ```text
    ...
    [WARNING] Tests run: 40, Failures: 0, Errors: 0, Skipped: 1
    [INFO] 
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    ...
   ```

## Basic Pitest Mutation Testing Setup
1. Add Pitest to your project. In the [`pom.xml`](./pom.xml) file, add the Pitest plugin and configure it to generate a coverage report during the test phase.
   Refer to the [Pitest quick start documentation](https://pitest.org/quickstart/maven/) for more information on Pitest configuration.
   <details>
      <summary>Solution</summary>   
      
      ```xml
      <plugin>
        <groupId>org.pitest</groupId>
        <artifactId>pitest-maven</artifactId>
        <version>1.6.7</version>
        <executions>
          <execution>
            <phase>test</phase>
            <goals>
              <goal>mutationCoverage</goal>
            </goals>
          </execution>
        </executions>
        <dependencies>
          <dependency>
            <groupId>org.pitest</groupId>
            <artifactId>pitest-junit5-plugin</artifactId>
            <version>0.14</version>
          </dependency>
        </dependencies>
        <configuration>
          <outputFormats>
            <format>HTML</format>
          </outputFormats>
        </configuration>
      </plugin>
      ```
   </details>
   
2. Add the Pitest JUnit 5 dependency
   <details>
      <summary>Solution</summary>

      ```xml
      <dependency>
        <groupId>org.pitest</groupId>
        <artifactId>pitest-junit5-plugin</artifactId>
        <version>0.14</version>
      </dependency>
      ```
   </details>
3. Run the unit tests again
   ```bash
   ./mvnw clean test
   ```
   Notice that after the unit tests run, the mutation tests will run.
   ```bash
    ...
    >> Line Coverage: 222/238 (93%)
    >> Generated 111 mutations Killed 75 (68%)
    >> Mutations with no coverage 8. Test strength 73%
    >> Ran 242 tests (2.18 tests per mutation)
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    ...
   ```
   Mutation tests can also be run separate from the test phase using the following command
   ```bash
   ./mvnw org.pitest:pitest-maven:mutationCoverage
   ```
4. Mutation testing results can be viewed in the [index.html](./target/pit-reports) file. Information on how to read this file can be found on slide 15 of the presentation deck.
   
   To find this file navigate to the `index.html` file found in `target/pit-reports/<timestamped folder>/index.html`

## Adding Target Classes
Target classes can be configured in the pom file so that only the desired classes are mutated during testing. Classes can be targeted individually or by folder. Classes that do not have unit tests or classes that would not be useful for mutation testing (i.e. configuration classes) can be left out to cut down on extraneous results and runtime.
1. Add the following target classes to the `configuration` section under `targetClasses`

    <details>
      <summary>Solution</summary>   

      ```xml
      <plugin>
        <groupId>org.pitest</groupId>
        <artifactId>pitest-maven</artifactId>
        <version>1.6.7</version>
        <executions>
          <execution>
            <phase>test</phase>
            <goals>
              <goal>mutationCoverage</goal>
            </goals>
          </execution>
        </executions>
        <dependencies>
          <dependency>
            <groupId>org.pitest</groupId>
            <artifactId>pitest-junit5-plugin</artifactId>
            <version>0.14</version>
          </dependency>
        </dependencies>
        <configuration>
          <targetClasses>
            <param>org.springframework.samples.petclinic.owner.Owner</param>
            <param>org.springframework.samples.petclinic.owner.OwnerController</param>
            <param>org.springframework.samples.petclinic.system.CrashController</param>
            <param>org.springframework.samples.petclinic.vet.VetController</param>
            <param>org.springframework.samples.petclinic.vet.Vet</param>
            <param>org.springframework.samples.petclinic.model*</param>
          </targetClasses>
          <outputFormats>
            <format>HTML</format>
          </outputFormats>
        </configuration>
      </plugin>
      ```
   </details>
    
2. Run the mutation testing goal again and notice the difference in runtime
    ```bash
    ./mvnw org.pitest:pitest-maven:mutationCoverage
    ```
   
## Killing A Surviving Mutation
1. In the Pitest report navigate to `org.springframework.samples.petclinic.owner > Owner.java`.
    * On line 102 we can see a mutation that survived
2. Click on the number `1` next to the line number `102`
    * In this case a call to `PropertyComparator.sort()` was removed and the unit test responsible for this line still passed
3. In the project navigate to the [ClinicServiceTests.java](./src/test/java/org/springframework/samples/petclinic/service/ClinicServiceTests.java) file in the `service` folder and find the `shouldFindSingleOwnerWithPet()` test.
4. Add an assertion to this test at the end that checks for properly sorted pets
    <details>
      <summary>Solution</summary>   

      ```
      @Test
	    void shouldFindSingleOwnerWithPet() {
            Owner owner = this.owners.findById(3);
            assertThat(owner.getLastName()).startsWith("Rodriquez");
            assertThat(owner.getPets()).hasSize(2);
            assertThat(owner.getPets().get(0).getType()).isNotNull();
            assertThat(owner.getPets().get(0).getType().getName()).isEqualTo("dog");
            assertThat(owner.getPets().get(0).getName()).isEqualTo("Jewel");
	    }
      ```
   </details>
5. Re-run the mutation testing goal and check the report. Notice that the mutation on line 102 is now killed.
   
    Note: you may need to run `./mvnw clean test` in order to see the new results

## SCM Mutation Coverage Analysis
Pitest can be configured to integrate with SCM in order to only mutate classes that contain the specified status of code (i.e. ADDED/MODIFIED). This is useful for saving time when developing locally by allowing you to quickly run mutation testing against newly written code.

1. Change the goal in the Pitest plugin to `scmMutationCoverage` and specify the status of the code to included as `MODIFIED,ADDED`

    <details>
      <summary>Solution</summary>   

      ```xml
      <plugin>
        <groupId>org.pitest</groupId>
        <artifactId>pitest-maven</artifactId>
        <version>1.6.7</version>
        <executions>
          <execution>
            <phase>test</phase>
            <goals>
              <goal>scmMutationCoverage</goal>
            </goals>
          </execution>
        </executions>
        <dependencies>
          <dependency>
            <groupId>org.pitest</groupId>
            <artifactId>pitest-junit5-plugin</artifactId>
            <version>0.14</version>
          </dependency>
        </dependencies>
        <configuration>
          <include>MODIFIED,ADDED</include>
          <targetClasses>
            <param>org.springframework.samples.petclinic.owner.Owner</param>
            <param>org.springframework.samples.petclinic.owner.OwnerController</param>
            <param>org.springframework.samples.petclinic.system.CrashController</param>
            <param>org.springframework.samples.petclinic.vet.VetController</param>
            <param>org.springframework.samples.petclinic.vet.Vet</param>
            <param>org.springframework.samples.petclinic.model*</param>
          </targetClasses>
          <outputFormats>
            <format>HTML</format>
          </outputFormats>
        </configuration>
      </plugin>
      ```
   </details>

2. Configure SCM with the repo URL at the project level of the pom file

    <details>
      <summary>Solution</summary>   

      ```xml
      <scm>
        <connection>scm:git:https://github.com/heidarianw/spring-petclinic.git</connection>
        <developerConnection>scm:git:https://github.com/heidarianw/spring-petclinic.git</developerConnection>
        <url>https://github.com/heidarianw/spring-petclinic.git</url>
      </scm>
      ```
   </details>

3. Add the Maven SCM plugin to the pom file
    <details>
      <summary>Solution</summary>   

      ```xml
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-scm-plugin</artifactId>
        <version>1.11.2</version>
        <configuration>
          <connectionType>connection</connectionType>
        </configuration>
      </plugin>
      ```
   </details>

4. Navigate to the [Person.java](./src/main/java/org/springframework/samples/petclinic/model/Person.java) file in the `model` folder and add a new method (Example found in solution below).
    <details>
      <summary>Solution</summary>

      ```
        @MappedSuperclass
        public class Person extends BaseEntity {
        
            @Column(name = "first_name")
            @NotEmpty
            private String firstName;
        
            @Column(name = "last_name")
            @NotEmpty
            private String lastName;
        
            public String getFirstName() {
                return this.firstName;
            }
        
            public void setFirstName(String firstName) {
                this.firstName = firstName;
            }
        
            public String getLastName() {
                return this.lastName;
            }
        
            public void setLastName(String lastName) {
                this.lastName = lastName;
            }
        
            public int getFirstNameLength() {
                return this.firstName.length();
            }
        
        }
      ```
   </details>

5. Run the `scmMutationCoverage` command
    ```bash
    ./mvnw org.pitest:pitest-maven:scmMutationCoverage
    ```
6. Navigate to the mutation testing report and notice that only the class with modified code was analized in the mutation testing analysis
