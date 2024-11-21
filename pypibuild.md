# build fasttext-numpy2 on pypi

note that building wheels for pypi on linux needs the
[manylinux](https://github.com/pypa/manylinux) project.

depending on the architecture you want to use,
you will need to find the corresponding docker image.

the 2 docker images used here end up creating the 
linux_2_28_x86_64 and linux2014_x86_64 wheels.

```bash
# clone and cd into
rm -rf dist/ wheelhouse/

pandoc --from=markdown --to=rst --output=python/README.rst python/README.md

# build and upload source dist
python -m build
python -m twine upload --repository pypi dist/fasttext-numpy2-*.tar.gz

# start each of the dockers and mount the repository inside
docker run -it --rm -v "$(pwd)":/workspace -w /workspace \
  --user "$(id -u):$(id -g)" quay.io/pypa/manylinux_2_28_x86_64
docker run -it --rm -v "$(pwd)":/workspace -w /workspace \
  --user "$(id -u):$(id -g)" quay.io/pypa/manylinux2014_x86_64

# inside docker build binary for various python versions
set -e
rm -rf dist/
for i in {6..13}; do
  echo build for python 3.${i}
  python3.${i} -m build
  auditwheel repair dist/fasttext_numpy2-*-cp3${i}-*.whl
done
rm -rf dist/

# repaired manylinux wheels are in wheelhouse/
# outside the docker upload them
python -m twine upload --repository pypi wheelhouse/*
```

