# Axis 1.4 internamente parcheado

Este proyecto utiliza una version internamente parcheada de Axis 1.4.

OWASP Dependency-Check identifica vulnerabilidades por coordenadas, nombre del artefacto y metadatos asociados. Por ese motivo puede seguir reportando CVEs correspondientes al Axis 1.4 original aunque el binario incluido en este proyecto contenga parches internos.

Las CVEs listadas en `dependency-check-suppressions.xml` fueron evaluadas y corregidas en la version parcheada incluida en este proyecto. La supresion no deshabilita el analisis de Axis en general: solo excluye las CVEs indicadas explicitamente para artefactos Axis.

## CVEs evaluadas

`CVE-2023-40743` describe el riesgo de usar `ServiceFactory.getService` con entradas no confiables, permitiendo mecanismos de lookup JNDI potencialmente peligrosos como LDAP y RMI.

Evidencia versionada:

- La rama actual contiene el commit upstream de parche `7e66753427466590d6def0125e448d2791723210`.
- `axis-rt-core/src/main/java/org/apache/axis/client/ServiceFactory.java` valida el valor `jndiName` antes de llamar a `InitialContext.lookup`.
- El codigo bloquea esquemas de lookup no soportados, incluyendo `LDAP`, `RMI`, `JMS`, `JMX`, `JRMP`, `JAVA`, `DNS`, `IIOP` y `CORBANAME`, y retorna `null` sin realizar el lookup.
- `CVE-2019-0227` queda cubierta por la misma validacion de mecanismos de lookup remotos no soportados.
- `CVE-2018-8032` describe XSS en la pagina de servicios del servlet. `axis-rt-core/src/main/java/org/apache/axis/transport/http/AxisServlet.java` codifica los nombres de servicios, URLs base y nombres de operaciones antes de escribirlos en HTML.
- `CVE-2012-5784` y `CVE-2014-3596` describen validacion incompleta del hostname TLS. `axis-rt-core/src/main/java/org/apache/axis/components/net/JSSESocketFactory.java` valida `subjectAltName` DNS y CN usando `LdapName`/`Rdn`, evitando parseo textual inseguro del subject.
- `CVE-2015-0897` no corresponde a Axis. Dependency-Check la reporta por un CPE falso (`line:line`) asociado a `axis-tools`; esa CVE corresponde a LINE para Android/iOS, no a Apache Axis ni a este fork.

## Uso con OWASP Dependency-Check

El repositorio incluye `dependency-check-suppressions.xml` en la raiz y el POM raiz configura `dependency-check-maven` para usarlo:

```xml
<configuration>
    <suppressionFiles>
        <suppressionFile>${project.basedir}/dependency-check-suppressions.xml</suppressionFile>
    </suppressionFiles>
</configuration>
```

Si el analisis se ejecuta desde un POM externo, por ejemplo `pom-check.xml`, ese POM tambien debe referenciar el archivo de supresiones:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.owasp</groupId>
            <artifactId>dependency-check-maven</artifactId>
            <version>12.1.9</version>
            <configuration>
                <suppressionFiles>
                    <suppressionFile>C:/Development/pruebas/axis-axis1-java-master/dependency-check-suppressions.xml</suppressionFile>
                </suppressionFiles>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Tambien puede evitarse la advertencia de Maven por `systemPath` hard-codeado declarando una propiedad en el POM externo:

```xml
<properties>
    <axis.jar.path>C:/Development/pruebas/axis-axis1-java-master/axis/target/axis-1.4-statum-p1.jar</axis.jar.path>
</properties>
```

y usando `${axis.jar.path}` en el `systemPath`.

Al entregar el binario al cliente, incluir:

- `dependency-check-suppressions.xml`
- `docs/security/axis-1.4-patched.md`
- `axis-1.4-statum-p1.jar`

Referencias:

- OWASP Dependency-Check suppression file: https://dependency-check.github.io/DependencyCheck/general/suppression.html
- NVD CVE-2023-40743: https://nvd.nist.gov/vuln/detail/CVE-2023-40743
- Commit upstream de parche: https://github.com/apache/axis-axis1-java/commit/7e66753427466590d6def0125e448d2791723210
