Contributed from the community.


**Amazon Linux AMI m4.xlarge**

sudo yum install emacs git git-core mod_dav_svn subversion freeglut freeglut-devel clang -y

sudo yum groupinstall "Development Tools" 

sudo yum groupinstall "Development Libraries"

**Install cmake**

sudo git clone https://cmake.org/cmake.git 

cd cmake

sudo ./bootstrap

sudo make

sudo make install

cd ..

mkdir bin

cd bin

**Install ANTs** 

git clone git://github.com/stnava/ANTs.git

mkdir antsbin

cd antsbin

ccmake ../ANTs

(type "c", “c” and then “g” )

sudo make -j 4

**Edit .bash_profile**

emacs .bash_profile

(added the following paths at the end of the file:)

export PATH=/home/ec2-user/bin/antsbin/bin:/home/ec2-user/bin/bin/ANTs/Scripts:$PATH

export ANTSPATH=/home/ec2-user/bin/antsbin/bin:/home/ec2-user/bin/bin/ANTs/Scripts/

export ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS=1

