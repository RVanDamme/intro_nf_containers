# Singularity and Nextflow a usage demo

Having all in one tools is great for analysis, but something better is to have automated workflows.
Why would we bother to run a tool wait for it to be done, run the second one, then the third one and so on, when we can simply compose them in a script that will do it for us?
Creating workflows is a long, annoying process when done in bash that requires some skills and that will be highly specific to the system it is supposed to run on. In addition, in today's science we have many challenges to face to provide the best science we can.

* Reproducibility Issues (*280 hours to reproduce a typical biology paper; https://doi.org/10.1371/journal.pone.0080278*);
* Portability (*what happens when I want to run on another system*);
* Scalability (*what happens if my system has a 1000 cpus now*);
* Usability (*can anyone run easily my workflow or do they need to spend hours reading the code and adapting to their system*);
* Consistency issues (*same analysis same system; same analysis different system*)

To solve all those, workflow management system and language have appeared amongst which [galaxy project](https://africa.usegalaxy.eu/), [Nextflow](https://www.nextflow.io/) and [Snakemake](https://snakemake.readthedocs.io/en/stable/)

We will use Nextflow today as it is one of the leading language, who works on every system and cloud platform natively, integrates the container management and has a large community support and activity.

The best example is the [NF-core](https://nf-co.re/) group, which is a community effort to collect a curated set of pipelines using nextflow.
They have extensive manual, guides and configuration making it the best to work on anykind of system.

Despite having all the computing compiled in a workflow, there is challenges remaining. Amongst which managing software and dependencies for all of your analysis is a complex and time consuming process.
Fortunately, people are developing "containers".

A container is a light weight "box" that contains the code and all the dependencies of an application.
It can be use in any system in a fast and reliable way as only the minimal requirement (code, runtime, system tools and library) are present in the container. To run a container a container manager is required whether it is Docker, Singularity, Podman, LXD or another one.

![Representation of containers on the system](https://www.docker.com/wp-content/themes/divi-child/assets/images/product/product-body-background.svg =250x250)

Physical shipping containers and software containers work by the same principles. Container ships are built to carry containers and anyone who package goods in a container according to shipping standards know that the container can be carried by any ship designed to carry them. 
In addition harbours are built to quickly load and unload containers from the ship and during transport each container protect the goods inside meaning that it doesn’t matter what is shipped alongside of it on the same ship.

Standardisation in containers also make bioinformatics portable and shareable. Imagine for example if all the equipment on this ship was stacked without containers and you then want to move it all by truck from the harbour, disentangling the pile and distributing it over the trucks would be a nightmare. With containers everyone know the specifications of carrying a containers and you can put each container on a truck (next pixture). With containerbased bioinformatics I can do the same thing. My laptop got all the software necessary to run Singularity containers, it is not as powerful as a big server but I can load a container, run the software inside of it and when I am done I can unload it and load the next container necessary for my workflow to run. 

[Singularity](https://docs.sylabs.io/guides/3.5/user-guide/introduction.html) is one of the container platflorm that allows you to create and run containers, if you want to search for containers to use I advise [Biocontainers](https://biocontainers.pro/) and [Dockerhub](https://hub.docker.com/) (all working with singularity).

## Singularity installation

User guide: https://docs.sylabs.io/guides/3.10/user-guide/
Admin guide: https://docs.sylabs.io/guides/latest/admin-guide/index.html 

### Linux and WSL

#### Install Dependencies

You must first install development tools and libraries to your host.

On Debian-based systems, including Ubuntu:

```sh
# Ensure repositories are up-to-date
sudo apt-get update
# Install debian packages for dependencies
sudo apt-get install -y \
    build-essential \
    libseccomp-dev \
    libglib2.0-dev \
    pkg-config \
    squashfs-tools \
    cryptsetup \
    runc
```

On CentOS/RHEL:

```sh
# Install basic tools for compiling
sudo yum groupinstall -y 'Development Tools'
# Install RPM packages for dependencies
sudo yum install -y \
    libseccomp-devel \
    glib2-devel \
    squashfs-tools \
    cryptsetup \
    runc
```

_Note - `runc` can be ommitted if you will not use the `singularity oci`
commands._

#### Install Go

Singularity is written in Go, and may require a newer version of Go than is
available in the repositories of your distribution. We recommend installing the
latest version of Go from the [official binaries](https://golang.org/dl/).

First, download the Go tar.gz archive to `/tmp`, then extract the archive to
`/usr/local`.

_**NOTE:** if you are updating Go from a older version, make sure you remove
`/usr/local/go` before reinstalling it._

```sh
export VERSION=1.19.3 OS=linux ARCH=amd64  # change this as you need

wget -O /tmp/go${VERSION}.${OS}-${ARCH}.tar.gz \
  https://dl.google.com/go/go${VERSION}.${OS}-${ARCH}.tar.gz
sudo tar -C /usr/local -xzf /tmp/go${VERSION}.${OS}-${ARCH}.tar.gz
```

Finally, add `/usr/local/go/bin` to the `PATH` environment variable:

```sh
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

#### Download SingularityCE from a release

You can download SingularityCE from one of the releases. To see a full list, visit the [GitHub release page](https://github.com/sylabs/singularity/releases). After deciding on a release to install, you can run the following commands to proceed with the installation.

```sh
export VERSION=3.10.4 && # adjust this as necessary

wget https://github.com/sylabs/singularity/releases/download/v${VERSION}/singularity-ce-${VERSION}.tar.gz 

tar -xzf singularity-ce-${VERSION}.tar.gz 

cd singularity-ce-${VERSION}
```

#### Compiling SingularityCE

You can configure, build, and install SingularityCE using the following
commands:

```sh
./mconfig

make -C builddir

sudo make -C builddir install
```

And that's it! Now you can check your SingularityCE version by running:

```sh
singularity --version
```

The `mconfig` command accepts options that can modify the build and installation
of SingularityCE. For example, to build in a different folder and to set the
install prefix to a different path:

```sh
./mconfig -b ./buildtree -p /usr/local
```

See the output of `./mconfig -h` for available options.

#### Test singularity

After installation you can perform a basic test of Singularity functionality by executing a simple container from the Sylabs Cloud library:

```sh
singularity exec library://alpine cat /etc/alpine-release
```
```sh
3.10.0
```

Or

```sh
singularity --debug run library://sylabsed/examples/lolcow
```

See the [user guide](https://www.sylabs.io/guides/3.10/user-guide/) for more information about how to use SingularityCE.
singularity buildcfg

Running `singularity buildcfg` will show the build configuration of an installed version of SingularityCE, and lists the paths used by SingularityCE. Use `singularity buildcfg` to confirm paths are set correctly for your installation, and troubleshoot any ‘not-found’ errors at runtime.

```sh
singularity buildcfg
```
```sh
PACKAGE_NAME=singularity
PACKAGE_VERSION=3.10.0
BUILDDIR=/home/dtrudg/Sylabs/Git/singularity/builddir
PREFIX=/usr/local
EXECPREFIX=/usr/local
BINDIR=/usr/local/bin
SBINDIR=/usr/local/sbin
LIBEXECDIR=/usr/local/libexec
DATAROOTDIR=/usr/local/share
DATADIR=/usr/local/share
SYSCONFDIR=/usr/local/etc
SHAREDSTATEDIR=/usr/local/com
LOCALSTATEDIR=/usr/local/var
RUNSTATEDIR=/usr/local/var/run
INCLUDEDIR=/usr/local/include
DOCDIR=/usr/local/share/doc/singularity
INFODIR=/usr/local/share/info
LIBDIR=/usr/local/lib
LOCALEDIR=/usr/local/share/locale
MANDIR=/usr/local/share/man
SINGULARITY_CONFDIR=/usr/local/etc/singularity
SESSIONDIR=/usr/local/var/singularity/mnt/session
```

Note that the LOCALSTATEDIR and SESSIONDIR should be on local, non-shared storage.

The list of files installed by a successful setuid installation of SingularityCE can be found in the [appendix, installed files section.](https://docs.sylabs.io/guides/latest/admin-guide/appendix.html#installed-files)


### Macos

SingularityCE is available via Vagrant (installable with Homebrew or manually)

To use Vagrant via Homebrew:

```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

brew install --cask virtualbox vagrant vagrant-manager
```
#### SingularityCE Vagrant Box

Open a terminal (Mac) and create and enter a directory to be used with your Vagrant VM.

```sh
mkdir vm-singularity-ce && \
    cd vm-singularity-ce
```

If you have already created and used this folder for another VM, you will need to destroy the VM and delete the Vagrantfile.

```sh
vagrant destroy && \
    rm Vagrantfile
```

Then issue the following commands to bring up the Virtual Machine. (Substitute a different value for the $VM variable if you like.)

```sh
export VM=sylabs/singularity-ce-3.8-ubuntu-bionic64 && \
    vagrant init $VM && \
    vagrant up && \
    vagrant ssh
```

You can check the installed version of SingularityCE with the following:

```sh
singularity version
```
```sh
3.10.0
```

Of course, you can also start with a plain OS Vagrant box as a base and then install SingularityCE using one of the above methods for Linux.

## Nextflow installation

Main website and installation: https://www.nextflow.io/ 
Documentation: https://www.nextflow.io/docs/latest/index.html


### Make sure Java 11 or later is installed on your computer by using the command:
```sh
java -version 
```

#### If it's not installed you can install it as following:

Ubuntu:

```sh
sudo apt-get install openjdk-11-jre-headless
```

Centos:

```sh
sudo yum install java-11-openjdk-devel
```

Mac:

##### Step 1: Install Homebrew (if you haven’t done it already)

```sh
/bin/bash -c “$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)”
```

##### Step 2: Update it (if you haven’t done it already)

```sh
brew update
```

##### Step 3: Install Java11

```sh
brew install java11
```

##### Step 4: Symlink it

If you skip this step the system won’t be able to find a java runtime for you to use.

```sh
sudo ln -sfn /usr/local/opt/openjdk@11/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-11.jdk
```

### Set up Nextflow

```sh
curl -s https://get.nextflow.io | bash

chmod +x nextflow
```

Optionally, move the `nextflow` file to a directory accessible by your `$PATH` variable (this is only required to avoid remembering and typing the full path to `nextflow` each time you need to run it).


### Testing

Run the classic Hello world by entering the following command:
```sh
./nextflow run hello
```

## Running your first nf-core pipeline

Since we have a time limit, we will run the genomeassembler pipeline. It is an **unreleased non finished** pipeline but simple enough that it should run on any computer as a demonstration pipeline.

https://nf-co.re/genomeassembler 

First we will read the main page and quick start, then read the usage doc and parameters and finally run the following command:

```sh
nextflow run nf-core/genomeassembler -r master -profile test,singularity --outdir <OUTDIR>
```

More advanced parameters:

```sh
nextflow run nf-core/genomeassembler -r master -profile test,singularity --outdir <OUTDIR> --max_cpus  --max_memory 10.GB
```

Or 

```sh
nextflow run nf-core/genomeassembler -r master -profile test,singularity --outdir <OUTDIR> -with-report report.html -with-timeline timeline.html
```

## More

Of course nf-core is not composed only of non-working incomplete pipelines.
Here is some exemple of what you can do:

* [nf-core/eager](https://nf-co.re/eager) for research in ancient DNA in craddle of humanity
* [nf-core/methylseq](https://nf-co.re/methylseq) for research in methylation site of cassava plant infected or not
* [nf-core/demultiplex](https://nf-co.re/demultiplex) for demultiplexing ngs Illumina data
* [nf-core/nanoseq](https://nf-co.re/nanoseq) for demultiplecing and QC and Alignment of Nanopore data
* [nf-core/rnaseq](https://nf-co.re/rnaseq) for expression in heatstroke cattle vs non heatstroke
* [nf-core/ampliseq](https://nf-co.re/ampliseq) for amplicon sequencing analysis
* [nf-core/mag](https://nf-co.re/mag) and soon to be updated MUFFIN for metagenomics
* [MUFFIN](https://github.com/RVanDamme/MUFFIN)


# Additional links and reading:
* All [nf-core pipelines](https://nf-co.re/pipelines)
* How to write your own nextflow pipeline a [complete course](https://training.seqera.io/)
* More [pipelines](https://github.com/nextflow-io/awesome-nextflow)


![A scientist getting zap after pushing a button will try again, a normal person will not](https://imgs.xkcd.com/comics/the_difference.png)
