Maven项目Docker部署插件



```xml
<plugin>
<groupId>com.spotify</groupId>
<artifactId>docker-maven-plugin</artifactId>
<version>1.0.0</version>
<configuration>
    <imageName>${docker.image.prefix}/${project.artifactId}		</imageName>
    <dockerDirectory>src/main/docker</dockerDirectory>
    <resources>
        <resource>
        <targetPath>/</targetPath>
        <directory>${project.build.directory}</directory>
        <include>${project.build.finalName}.jar</include>
        </resource>
    </resources>
</configuration>
</plugin>
```

