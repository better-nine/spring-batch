<img align="right" src="spring-batch-docs/modules/ROOT/assets/images/spring-batch.png" width="200" height="200">

# Spring Batch [![build status](https://github.com/spring-projects/spring-batch/actions/workflows/continuous-integration.yml/badge.svg)](https://github.com/spring-projects/spring-batch/actions/workflows/continuous-integration.yml)

Spring Batch는 엔터프라이즈 시스템의 일상적인 운영에 필수적인 강력한 배치 애플리케이션 개발을 지원하도록 설계된 가볍고 포괄적인 배치 프레임워크입니다.
Spring Batch는 사람들이 [Spring Framework](https://github.com/spring-projects/spring-framework)에서 알게 된 생산성, POJO 기반 개발 접근 방식 및 일반적인 사용 편의성 기능을 기반으로 구축되었습니다. 개발자는 필요할 때 더욱 발전된 엔터프라이즈 서비스에 쉽게 액세스하고 활용할 수 있습니다.

Batch 애플리케이션을 위한 런타임 조정 도구를 찾고 있거나 현재 및 기록 실행을 보기 위한 관리 콘솔이 필요한 경우 [Spring Cloud Data Flow] (https://cloud.spring.io/spring-cloud)을 살펴보세요. 
Spring Batch 애플리케이션을 포함한 데이터 통합 ​​기반 마이크로서비스를 배포하고 실행하기 위한 오케스트레이션 도구입니다.


# 시작하기

## 2분 과정

이 신속한 과정에선 Spring Batch를 사용하여 간단한 배치 작업을 실행하기 위해 최소 프로젝트를 설정하는 방법을 보여줍니다.

선호하는 IDE에서 새로운 Maven 기반 Java 17+ 프로젝트를 생성하고 다음 종속성을 추가합니다.:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.batch</groupId>
        <artifactId>spring-batch-core</artifactId>
        <version>${LATEST_VERSION}</version>
    </dependency>
    <dependency>
        <groupId>org.hsqldb</groupId>
        <artifactId>hsqldb</artifactId>
        <version>${LATEST_VERSION}</version>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

다음으로 job repository에서 사용할 데이터 소스 및 트랜잭션 관리자를 정의하는 configuration class를 만듭니다.:

```java
import javax.sql.DataSource;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.support.JdbcTransactionManager;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseBuilder;

@Configuration
public class DataSourceConfiguration {

	@Bean
	public DataSource dataSource() {
		return new EmbeddedDatabaseBuilder()
			.addScript("/org/springframework/batch/core/schema-hsqldb.sql")
			.build();
	}

	@Bean
	public JdbcTransactionManager transactionManager(DataSource dataSource) {
		return new JdbcTransactionManager(dataSource);
	}

}
```

In this tutorial, an embedded [HSQLDB](http://www.hsqldb.org) database is created and initialized with Spring Batch's meta-data tables.

Finally, create a class to define the batch job:

```java
import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobParameters;
import org.springframework.batch.core.Step;
import org.springframework.batch.core.configuration.annotation.EnableBatchProcessing;
import org.springframework.batch.core.job.builder.JobBuilder;
import org.springframework.batch.core.launch.JobLauncher;
import org.springframework.batch.core.repository.JobRepository;
import org.springframework.batch.core.step.builder.StepBuilder;
import org.springframework.batch.repeat.RepeatStatus;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.jdbc.support.JdbcTransactionManager;

@Configuration
@EnableBatchProcessing
@Import(DataSourceConfiguration.class)
public class HelloWorldJobConfiguration {

	@Bean
	public Step step(JobRepository jobRepository, JdbcTransactionManager transactionManager) {
		return new StepBuilder("step", jobRepository).tasklet((contribution, chunkContext) -> {
			System.out.println("Hello world!");
			return RepeatStatus.FINISHED;
		}, transactionManager).build();
	}

	@Bean
	public Job job(JobRepository jobRepository, Step step) {
		return new JobBuilder("job", jobRepository).start(step).build();
	}

	public static void main(String[] args) throws Exception {
		ApplicationContext context = new AnnotationConfigApplicationContext(HelloWorldJobConfiguration.class);
		JobLauncher jobLauncher = context.getBean(JobLauncher.class);
		Job job = context.getBean(Job.class);
		jobLauncher.run(job, new JobParameters());
	}

}
```

The job in this tutorial is composed of a single step that prints "Hello world!" to the standard output.

You can now run the `main` method of the `HelloWorldJobConfiguration` class to launch the job. The output should be similar to the following:

```
INFO: Finished Spring Batch infrastructure beans configuration in 8 ms.
INFO: Starting embedded database: url='jdbc:hsqldb:mem:testdb', username='sa'
INFO: No database type set, using meta data indicating: HSQL
INFO: No Micrometer observation registry found, defaulting to ObservationRegistry.NOOP
INFO: No TaskExecutor has been set, defaulting to synchronous executor.
INFO: Job: [SimpleJob: [name=job]] launched with the following parameters: [{}]
INFO: Executing step: [step]
Hello world!
INFO: Step: [step] executed in 10ms
INFO: Job: [SimpleJob: [name=job]] completed with the following parameters: [{}] and the following status: [COMPLETED] in 25ms
```

## Getting Started Guide

This guide is a more realistic tutorial that shows a typical ETL batch job that reads data from a flat file, transforms it and writes it to a relational database.
It is a Spring Batch project based on Spring Boot. You find the Getting Started Guide here: [Creating a Batch Service](https://spring.io/guides/gs/batch-processing/).

## Samples

You can find several samples to try out here: [Spring Batch Samples](https://github.com/spring-projects/spring-batch/tree/main/spring-batch-samples).

# Getting Help

If you have a question or a support request, please open a new discussion on [GitHub Discussions](https://github.com/spring-projects/spring-batch/discussions)
or ask a question on [StackOverflow](https://stackoverflow.com/questions/tagged/spring-batch).

Please do **not** create issues on the [Issue Tracker](https://github.com/spring-projects/spring-batch/issues) for questions or support requests.
We would like to keep the issue tracker **exclusively** for bug reports and feature requests.

# Reporting issues

Spring Batch uses [GitHub Issues](https://github.com/spring-projects/spring-batch/issues) to record bugs and feature requests. If you want to raise an issue, please follow the recommendations below:

* Before you open an issue, please search the issue tracker to see if someone has already reported the problem. If the issue doesn't already exist, create a new issue.
* Please provide as much information as possible in the issue report by following the [Issue Reporting Template](https://github.com/spring-projects/spring-batch/blob/main/.github/ISSUE_TEMPLATE/bug_report.md).
* If you need to paste code or include a stack trace, please use Markdown escapes (```) before and after your text.

For non trivial bugs, please create a test case or a project that replicates the problem and attach it to the issue, as detailed in the [Issue Reporting Guidelines](https://github.com/spring-projects/spring-batch/blob/main/ISSUE_REPORTING.md).

# Reporting Security Vulnerabilities

Please see our [Security policy](https://github.com/spring-projects/spring-batch/security/policy).

# Building from Source

## Using the Command Line

Clone the git repository using the URL on the Github home page:

    $ git clone git@github.com:spring-projects/spring-batch.git
    $ cd spring-batch

Maven is the build tool used for Spring Batch. You can build the project with the following command:

    $ ./mvnw package

If you want to perform a full build with all integration tests, then run:

    $ ./mvnw verify

Please note that some integration tests are based on Docker, so please make sure to have Docker up and running before running a full build.

To generate the reference documentation, run the following commands:

```
$ cd spring-batch-docs
$ ../mvnw antora:antora
```

The reference documentation can be found in `spring-batch-docs/target/anotra/site`.

## Using Docker

If you want to build the project in a Docker container, you can proceed as follows:

```
$> docker run -it --mount type=bind,source="$(pwd)",target=/spring-batch maven:3-openjdk-17 bash
#> cd spring-batch
#> ./mvnw package
```

This will mount the source code that you cloned previously on the host inside the container.
If you want to work on a copy of the source code inside the container (no side effects on the host),
you can proceed as follows:

```
$> docker run -it maven:3-openjdk-17 bash
#> git clone https://github.com/spring-projects/spring-batch.git
#> cd spring-batch
#> ./mvnw package
```

# Contributing to Spring Batch

We welcome contributions in any kind! Here are some ways for you to contribute to the project:

* Get involved with the Spring Batch community on [Twitter](https://twitter.com/springbatch), [GitHub Discussions](https://github.com/spring-projects/spring-batch/discussions) and [StackOverflow](https://stackoverflow.com/questions/tagged/spring-batch) by responding to questions and joining the debate.
* Create [issues](https://github.com/spring-projects/spring-batch/issues) for bugs and new features or comment and vote on the ones that you are interested in.
* Help us reproduce issues marked with [status: need-help-to-reproduce](https://github.com/spring-projects/spring-batch/labels/status%3A%20need-help-to-reproduce) by following the [Issue Reporting Guidelines](https://github.com/spring-projects/spring-batch/blob/main/ISSUE_REPORTING.md).
* Github is for social coding: if you want to write code, we encourage contributions through pull requests. If you want to contribute code this way, please familiarize yourself with the process outlined here: [Contributor Guidelines](https://github.com/spring-projects/spring-batch/blob/main/CONTRIBUTING.md).
* Watch for Spring Batch related articles on [spring.io](https://spring.io).

Before we accept pull requests, we will need you to sign the [contributor's agreement](https://support.springsource.com/spring_committer_signup).  Signing the contributor's agreement does not grant anyone commit rights to the main repository, but it does mean that we can accept your contributions, and you will get an author credit if we do.  Active contributors might be asked to join the core team, and given the ability to merge pull requests.

# Code of Conduct

Please see our [code of conduct](https://github.com/spring-projects/.github/blob/main/CODE_OF_CONDUCT.md).

# License

Spring Batch is Open Source software released under the [Apache 2.0 license](https://www.apache.org/licenses/LICENSE-2.0.html).
