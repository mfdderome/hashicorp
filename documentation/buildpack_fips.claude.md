# Me

Is it possible to build FIPS image ready for openshift using paketo?

# Claude

Here's a thorough breakdown of the situation:

---

## Building FIPS-Ready Images for OpenShift with Paketo

The short answer is: **yes, it's possible, but with important nuances depending on your path.**

### 1. Tanzu Buildpacks (VMware/Broadcom): FIPS Stack Exists

The most direct FIPS support in the Paketo ecosystem comes from **VMware Tanzu Buildpacks**, which is the commercially supported flavor built on top of Paketo. Tanzu Buildpacks documentation explicitly lists a "Base FIPS Stack for Ubuntu 22.04" alongside the standard Base, Full, Tiny, and Static stacks. This is available for buildpacks like Spring Boot, Apache Tomcat, and others.

If you're using **Tanzu Application Platform (TAP)** or **Tanzu Build Service (TBS)** on OpenShift, this is the most supported path to FIPS-compliant images.

---

### 2. Community Paketo (Open Source): No Official FIPS Builder

The open-source Paketo project does **not** ship a dedicated FIPS builder. The Paketo project currently officially supports Ubuntu 22.04 (Jammy Jellyfish) in Tiny, Base, and Full stack variants that differ in the number of packages installed. None of these are FIPS-hardened out of the box.

---

### 3. The UBI + Paketo Path (Red Hat's Approach for OpenShift)

Red Hat has been working on a different FIPS-compatible path for OpenShift: using **Red Hat UBI (Universal Base Images)** as the stack base with Paketo buildpacks. Red Hatters worked with the CNCF community to extend the buildpack specification to enable UBI builder images, creating extensions for Node.js and Java, as well as the `ubi-base-stack` and `builder-ubi-base`.

This is significant for FIPS because UBI images are based on RHEL, which has FIPS-validated cryptographic modules. However, this is still working through the hardening process as the image extension feature is still experimental in the specification.

---

### 4. Known FIPS + Paketo Issue on OpenShift

There is a documented compatibility issue worth knowing about: in older OpenShift Serverless versions, Springboot function builds using Paketo buildpacks failed when FIPS mode was enabled on the cluster, producing errors like "buildpack API version '0.7' is incompatible with the lifecycle." This has since been resolved in newer versions, but it signals that you should verify version compatibility carefully.

---

### Your Options in Practice

| Path | FIPS Stack | OpenShift Ready | Notes |
|---|---|---|---|
| **Tanzu Buildpacks** | ✅ Yes (Base FIPS Stack) | ✅ Yes | Commercial product; best supported path |
| **Paketo + UBI Base** | Partially (RHEL crypto) | ✅ Yes | Experimental image extensions; Red Hat-backed |
| **Paketo + Custom FIPS Stack** | 🔧 DIY | ✅ Yes | Build your own FIPS-enabled stack via custom Dockerfile |
| **Open-source Paketo Jammy** | ❌ No | ✅ Yes | Ubuntu; not FIPS-validated |

For the **DIY approach**, Paketo supports creating a custom stack by modifying a base Paketo Dockerfile to add your desired packages (like FIPS-related OpenSSL), then building a custom stack and pointing a custom builder to it.

---

### Recommendation

If you need fully supported FIPS on OpenShift, the best current path is either **Tanzu Buildpacks with the FIPS stack** or the **Paketo + UBI builder** route being championed by Red Hat. The UBI path is the more natural fit for OpenShift specifically since it leverages RHEL's FIPS-certified crypto, though you should be aware it's still maturing. Would you like more detail on any of these approaches?

# Me

Does Tanzu Buildpacks by vmware broadcom uses redhat ubi images?

# Copilot

Yes, but it's **opt-in and limited**. Here's the clear picture from the official Broadcom/VMware docs:

### Default stack: Ubuntu, not UBI

Tanzu Buildpacks primarily provide stacks based on Ubuntu 22.04 (Jammy Jellyfish) and Ubuntu 18.04 (Bionic Beaver). Ubuntu Jammy is the default as of the latest Tanzu Application Platform release.

### UBI support: available but optional

A UBI builder does exist — it uses Red Hat Universal Base Image (UBI) v8 for both build and run images. However, it only supports Java and Node.js. To enable it, you must explicitly set `enable_ubi_builder: true` in your `tbs-full-deps-values.yaml`, and it requires the full dependencies profile.

### Key limitations of the UBI builder

The UBI HTTPD dependency does not come packaged with some of the additional libraries that are part of the Ubuntu package, because UBI 8 does not support them — meaning some HTTPD workflows that run on Ubuntu will not run on UBI.

### Summary table

| Stack | Base OS | Default? | Languages Supported |
|---|---|---|---|
| Jammy Tiny/Base/Full | Ubuntu 22.04 | ✅ Yes | All |
| UBI 8 Standard | Red Hat UBI 8 | ❌ No (opt-in) | Java, Node.js only |
| Jammy FIPS Base | Ubuntu 22.04 (FIPS) | ❌ No (opt-in) | Most |

### Bottom line for your FIPS + OpenShift use case

This means that if you need **UBI + FIPS** specifically, the Tanzu UBI builder alone doesn't give you that — it's UBI 8 but without an explicit FIPS guarantee. For a fully FIPS-validated stack on OpenShift, the combination you'd want is UBI 8 (which uses RHEL's FIPS-certified OpenSSL) enabled via `enable_ubi_builder: true`, but you'd be limited to Java or Node.js workloads. For other languages, you'd need to fall back to the Ubuntu Jammy FIPS stack or build a custom stack.

# Me

What about paketo community?

# Copilot

Great news here — the Paketo community has been actively progressing UBI support. Here's the full picture:

### UBI support is now official in Paketo community

The Paketo community has been working on UBI support over the last few years after agreeing on the direction. Typically support for new components starts in the `paketo-community` organization and then is moved to the `paketo-buildpacks` organization once it matures — and that graduation has now happened.

### Available UBI builders

Two official builders exist in the `paketo-buildpacks` org:

**UBI 8 (buildpackless base):** This builder uses the UBI 8 Base Stack and contains no buildpacks nor order groups. To use it, you must specify buildpacks and extensions at build time. For example with `pack` CLI:
```
pack build nodejs-app \
  --path ./path/to/nodejs/app \
  --buildpack docker.io/paketobuildpacks/nodejs \
  --extension docker.io/paketobuildpacks/ubi-nodejs-extension \
  --builder docker.io/paketobuildpacks/builder-ubi8-buildpackless-base
```


**UBI 10 (newer):** There's also a `ubi-10-builder` that uses UBI 10 base images. Like UBI 8, it is buildpackless and requires you to specify buildpacks and extensions at build time.

### The extension mechanism — why it's "buildpackless"

The original challenge was that buildpacks would not allow RPM-distributed runtimes to be installed from a buildpack, and creating a builder image for each runtime and version would quickly lead to an unmaintainable forest of builders. The solution was CNB **image extensions**, which allow buildpacks to modify the build environment via `dnf`/`yum` at build time — something previously not permitted.

### Running on OpenShift

You can build and push directly to OpenShift using the community UBI builder with:
```
pack build <public-image-repository-url> \
  --docker-host=inherit \
  --builder paketocommunity/builder-ubi-base \
  --publish
```


### Caveats

- **Language support is still limited**: Java and Node.js extensions exist; other languages are not yet covered.
- **Extensions are experimental**: The image extension feature in the CNB spec is still considered experimental, so you need to enable experimental features in the `pack` CLI.
- **No FIPS-specific stack**: The UBI builders leverage UBI's RHEL-backed crypto (which is FIPS-validated on RHEL), but there is no explicit "FIPS mode" toggle in the Paketo UBI builder itself — you'd rely on the underlying UBI image's FIPS compliance and ensure your OpenShift cluster is running in FIPS mode.

So for your OpenShift + FIPS use case, the **Paketo community UBI 8 or UBI 10 builder** is the most natural open-source path, as long as you're working with Java or Node.js.