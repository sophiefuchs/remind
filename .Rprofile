local({
# setting RENV_PATHS_LIBRARY ensures packages are installed into renv/library
# for some reason this also has implications for symlinking into the global cache
Sys.setenv(RENV_PATHS_LIBRARY = "renv/library")

# remind's renv integration previously relied on renv.lock, but now it should generally not be used anymore
# this can safely be removed in January 2024
if (file.exists("renv.lock") && file.exists("README.md") && !file.exists("renv/old_renv.lock")) {
  file.rename("renv.lock", "renv/old_renv.lock")
  message("moved legacy renv.lock to renv/old_renv.lock")
}

source("renv/activate.R")

renvVersion <- "0.16.0"
if (packageVersion("renv") != renvVersion) {
  renvLockExisted <- file.exists(renv::paths$lockfile())
  renv::upgrade(version = renvVersion, reload = TRUE, prompt = FALSE)
  if (!renvLockExisted) {
    unlink(renv::paths$lockfile())
  }
}

if (!"https://rse.pik-potsdam.de/r/packages" %in% getOption("repos")) {
  options(repos = c(getOption("repos"), pik = "https://rse.pik-potsdam.de/r/packages"))
}

# bootstrapping, will only run once after remind is freshly cloned
if (isTRUE(rownames(installed.packages(priority = "NA")) == "renv")) {
  message("R package dependencies are not installed in this renv, installing now...")
  renv::install("rmarkdown", prompt = FALSE) # rmarkdown is required to find dependencies in Rmd files
  renv::hydrate() # auto-detect and install all dependencies
  message("Finished installing R package dependencies.")
}

# bootstrapping python venv, will only run once after remind is freshly cloned
if (!dir.exists(".venv/")
    && (Sys.which("python3") != ""
        || (Sys.which("python.exe") != ""
            && suppressWarnings(isTRUE(startsWith(system2("python.exe", "--version", stdout = TRUE), "Python 3")))
           ))) {
  message("Python venv is not available, setting up now...")
  # use system python to set up venv
  if (.Platform$OS.type == "windows") {
    system2("python.exe", c("-mvenv", ".venv"))
    pythonInVenv <- normalizePath(file.path(".venv", "Scripts", "python.exe"), mustWork = TRUE)
  } else {
    system2("python3", c("-mvenv", ".venv"))
    pythonInVenv <- normalizePath(file.path(".venv", "bin", "python"), mustWork = TRUE)
  }
  # use venv python to install dependencies in venv
  system2(pythonInVenv, c("-mpip", "install", "--upgrade", "pip", "wheel"))
  system2(pythonInVenv, c("-mpip", "install", "-r", "requirements.txt"))
}

# Configure locations of REMIND input data
# These can be located in directories on the local machine, remote directories,
# or default directories on the cluster.
# To use these, set the environment variable in your ~/.bashrc file in your home
# direcotry (on linux) or in the system environment variables dialog (on windows):

# local directories
# e.g.
# on Linux (separate multiple paths by colons)
# REMIND_repos_dirs="/my/first/path:/my/second/path"
# on Windows (separate multiple paths by semicolons)
# REMIND_repos_dirs="C:\my\first\path;D:\my\second\path"
remindReposDirs <- Sys.getenv("REMIND_repos_dirs")

# for scp targets, you need to set three environment variables
# on linux e.g. (separate multiple paths by semicolons)
# REMIND_repos_scp="scp://cluster.pik-potsdam.de/p/projects/rd3mod/inputdata/output;scp://cluster.pik-potsdam.de/p/projects/remind/inputdata/CESparametersAndGDX"
# REMIND_repos_scp_user="myusername"  # use your user name on the scp target, e.g. the cluster
# REMIND_repos_scp_key="/home/myusername/.ssh/id_ed25519"  # path to your your ssh private key on your laptop
# on windows e.g.
# REMIND_repos_scp="scp://cluster.pik-potsdam.de/p/projects/rd3mod/inputdata/output;scp://cluster.pik-potsdam.de/p/projects/remind/inputdata/CESparametersAndGDX"
# REMIND_repos_scp_user="myusername"  # use your user name on the scp target, e.g. the cluster
# REMIND_repos_scp_key="C:\Users\myusername\.ssh\id_ed25519"  # path to your your ssh private key on your laptop
remindReposSCP <- Sys.getenv("REMIND_repos_scp") # scp URL
remindReposSCPUser <- Sys.getenv("REMIND_repos_scp_user")  # ssh user name
remindReposSCPKey <- Sys.getenv("REMIND_repos_scp_key")  # ssh key path

# unless specified otherwise, use cluster defaults
use_cluster_defaults <- TRUE

# add local directories, if any
if ("" != remindReposDirs) {
  directories <- unlist(strsplit(remindReposDirs, .Platform$path.sep,
                                 fixed = TRUE))
  directoriesList <- rep(list(NULL), length(directories))
  names(directoriesList) <- directories

  options(remind_repos = c(options("remind_repos")[[1]], directoriesList))
  use_cluster_defaults <- FALSE
}

# add remote directories, if any remote directory and username and SSH key are set
if ("" != remindReposSCP && "" != remindReposSCPUser && "" != remindReposSCPKey) {
  SCPUrls <- unlist(strsplit(remindReposSCP, ";", fixed = TRUE))
  config <- list(list(username = remindReposSCPUser, ssh_private_keyfile = remindReposSCPKey))
  for (SCPUrl in SCPUrls) {
    names(config) <- SCPUrl
    options(remind_repos = c(options("remind_repos")[[1]], config))
  }
  use_cluster_defaults <- FALSE
}

# default to cluster directories
if (use_cluster_defaults &&
    all(file.exists(c("/p/projects/rd3mod/inputdata/output",
                      "/p/projects/remind/inputdata/CESparametersAndGDX")))) {
  options(remind_repos = list(
    "/p/projects/rd3mod/inputdata/output" = NULL,
    "/p/projects/remind/inputdata/CESparametersAndGDX" = NULL))
}

})
