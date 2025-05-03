# Deployment

Deploy proper version with activated profile (Java-8 or Java-11), dependency versions are defined by current branch.

``mvn -PJava-8 deploy``

JDK version specific dependency version properties have `-versionX` suffix.

All changes should be propagated from `master` branch to respective version branches to ease merging.