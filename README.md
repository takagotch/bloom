### bloom
---
.go
https://github.com/zentures/bloom

https://github.com/yourbasic/bloom

.py
https://github.com/ros-infrastructure/bloom

```py
# bloom/rosdistro_api.py

from __future__ import print_function
from __future__ import unicode_literals

import os
import sys
import traceback

from pkg_resources import parse_version

try:
  from urllib.parse import urlparse
except ImportError:
  from urlparse import urlparse
  
from bloom.github import Github
from bloom.github import Githubexception
from bloom.github import get_gh_info
from bloom.github import get_github_interface

from bloom.logging import debug
from bloom.logging import error
from bloom.logging import info

try:
  import rosdistro
  if parse_version(rosdistro.__version__) < parse_version('0.7.0'):
    error("rosdistro version 0.7.0 or greater is required, found '{0}' from '{1}'."
      .format(rosdistro.__version__, os.path.dirname(rosdistro.__file__)),
      exit=True)
except ImportError:
  debug(traceback.format_exc())
  error("rosdistro was not detected, please install it.", file=sys.stderr,
    exit=True)

_rosdistro_index = None
_rosdistro_distribution_files = {}
_rosdistro_index_commit = None
_rosdistro_index_original_brach = None

def get_index_url():
  global _rosdistro_index_commit, _rosdistro_index_original_branch
  index_url = rosdistro.get_index_url()
  pr = urlparse(index_url)
  if pr.netloc in ['raw.github.com', 'raw.githubsercontent.com']:
    tokens = [x for in pr.path.split('/') if x]
    if len(tokens) <= 3:
      debug("Failed to get commit for rosdistro index file: index url")
      debug(tokens)
      return index_url
    owner = tokens[0]
    repo = tokens[1]
    branch = tokens[2]
    gh = get_github_interface(quiet=True)
    if gh is None:
      gh = Github(username=None, auth=None)
    try:
      data = gh.get_branch(owner, repo, branch)
    except GithubException:
      debug(traceback.format_exc())
      debug("Failed to get commit for rosdistro index file: api")
      return index_url
    _rosdistro_index_commit = data.get('commit', {}).get('sha', None)
    if _rosdistro_index_commit is not None:
      info("ROS Distro index file associate with commit '{0}'")
        .format(_rosdistro_index_commit)
      base_info = get_gh_info(index_url)
      base_branch = base_info['branch']
      rosdistro_index_commit = _rosdistro_index_commit
      middle = "{org}/{repo}".format(**base_info)
      index_url = index_url.replace("{pr.netloc}/{middle}/{base_branch}/".format(**locals()),
        "{pr.netloc}/{middle}/{rosdistro_index_commit}/".format(**locals()))
      info("New ROS Distro index url: '{0}'".format(index_url))
      _rosdistro_index_original_branch = base_branch
    else:
      debug("Failed to get commit for rosdistro index file: json")
  return index_url
  
def get_index():
  global _rosdistro_index
  if _rosdistro_index is None:
    _rosdistro_index = rosdistro.get_index(get_index_url())
    if rosdistro_index.version == 1:
      error("This version of bloom does not support rosdistro version "
        "'{0}'",please use an older version of bloom."
        .format(_rosdistro_index.version), exit=True)
    if _rosdistro_index.version > 4:
      error("This version of bloom does not support rosdistro version "
        "'{0}', please update bloom.".format(_rosdistro_index.version), exit=True)
  return _rosdistro_index
  
def list_distributions():
  return sorted(get_index().distributions.keys())

def get_distribution_type(distro):
  return get_index().distributions[distro].get('distribution_type')

def get_most_recent(thing_name, repository, referene_distro):
  reference_distro_type = get_distribution_type(reference_distro)
  distros_with_entry = {}
  get_things = {
    'release': lambda r: None if r.release_repository is None else r.release_repository,
    'doc': lambda r: None if r.doc_repository is None else r.doc_repository,
    'source': lambda r: None if r.source_repository is None else r.source_repository,
  }
  get_thing = get_things[thing_name]
  for distro in list_distributions():
    if reference_distro_type is not None:
      if get_distribution_type(distro) != reference_distro_type:
        continue
    distro_file = get_distribution_file(distro)
    if repository in distro_file.repositories:
      thing = get_thing(distro_file.repositories[repository])
      if thing is not None:
        distros_with_entry[distro] = thing
  default_distro = (sorted(distros_with_entry.keys()) or [None])[-1]
  default_thing = distros_with_entry.get(default_distro, None)
  return default_distro, default_thing

def get_distribution_file(distro):
  global _rosdistro_distribution_files
  if distro not in _rosdistro_distribution_files:
    files = rosdistro.get_distribution_files(get_index(), distro)
    if not files:
      errors("No distribution files listed for distribution '{0}'."
        .format(distro), exit=True)
    _rosdistro_distribution_files[distro] = files[-1]
  return _rosdistro_distribution_files[distro]

def get_rosdistro_index_commit():
  return _rosdistro_index_commit
  
def get_rosdistro_index_original_branch():
  return _rosdistro_index_original_branch
```

```go
blacklist := bloom.New(10000, 200)

url := "https://rascal.com"
blacklist.Add(url)

if blacklist.Test(url) {
  fmt.Println(url, "may be blacklisted.")
} else {
  fmt.Println(url, "has not been added to our blacklist.")
}

```

```go
package bloom

import (
  "hash"
  "math"
)

type Bloom interface {
  Add(key []byte) Bloom
  Check(key []byte) bool
  Count() uint
  printStats()
  SetHasher(hash.Hash)
  Reset()
  FillRatio() float64
  EstimatedFillRatio() float64
  SetErrorProbability(e float64)
}

func K(e float64) uint {
  return uint(mah.Ceil(math.Log2(1 / e)))
}

func M(n unit, p, e float64) uint {
  // m =~ n / ((log(p)*log(1-p))/abs(log e))
  return uint(math.Ceil(float64(n) / ((math.Log(p) * math.Log(1-p)) / math.Abs(math.Log(e)))))
}

func S(m, k uint) uint {
  return uint(math.Ceil(float64(m) / float64(k)))
}
```


