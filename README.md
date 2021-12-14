# mustRead
Some useful links

FRONT WEB
https://domevents.dev/
stub dans le browser: https://requestly.io/

DDD
https://github.com/kgrzybek/modular-monolith-with-ddd

SOLID
https://qpautrat.fr/2021/04/26/solid-par-l-exemple/

DATA LBC
https://medium.com/leboncoin-engineering-blog/cooling-down-hot-data-from-kafka-to-athena-5918a628bd98


Test design
https://martinfowler.com/bliki/BeckDesignRules.html

JS basics : 
https://wesbos.com/javascript

Web Perf by url :
https://developers.google.com/web/tools/lighthouse




Vidéo éducative :

https://youtu.be/w7y-1eY0mcE

Vrac code to clean :
# bash intructions

# get cypress spec files by machine (parallelization)
function get_spec_files(){
  # you can uncomment echo instructions to check what the script does;
  # with for example; get_spec_files 3 0 "test"

  local parallelizationCounter=${1}
  local index=${2}
  local ft=${3}

  # check there are not more indexes than parallelizationCounter
  if [ $((index + 1)) -gt "$parallelizationCounter" ]; then
      echo "There are too many indexes compare to parallelizationCounter, check it"
      exit 1
  fi

  # manage launching of all tests
  if [[ $ft = "transverse" || $ft = "framework" ]]; then
#    echo "Transverse or framework launching, so we set empty value to get every tests"
    ft=""
    # toto/tutu are on others domain
    exclude="-not -path *ft_tutu/* -not -path *ft_toto/* -not -path *tete/*"
  else
    # replace '-' from git cz conf by '_' for folder declaration (must match .cz-config.cjs)
    ft=$(echo "$ft" | tr - _)
    # we set the full folder name
    ft=ft_"$ft"
  fi

  # find every spec.js files in folder cypress except E2E, create spec.txt with the result
  find cypress/**/"$ft" -name "*.spec.*" -not -path "*E2E/*" -not -path "*pending/*" $exclude > spec

  # check if file generated is empty; if it is, then exit because we found no test to run
  if [[ ! -s spec ]]; then
        echo "No tests found with '$ft', exiting. It must match the .cz-config.cjs declaration, replacing
    '-' by '_' for folder name. Check the find command if needed"
    exit 1
#  else
#    echo "From common.sh (get_spec_files), We found tests to run ----"
  fi

  # get the number of test files (occurrences of 'spec.' in file spec generated)
  specFilesNumber=$(grep -c "spec." spec)
  # parallelize per job
  filesPerJob=$(( specFilesNumber / parallelizationCounter ))
  modulo=$(( specFilesNumber % parallelizationCounter ))
  # don't forget modulo
  filesPerJobSplit=$(( filesPerJob + modulo ))
  # split per lines into X files (=$parallelizationCounter) named index0{0..X};
  split -l $filesPerJobSplit spec --numeric-suffixes index
  indexFileName="index0$index"


  if [ ! -e "$indexFileName" ]; then
      echo "File '$indexFileName' doesn't exist, not enough tests for parallelization, so exiting"
      exit 0
#  else
#      echo "File '$indexFileName' exists, we are getting spec files ---*"
  fi

  # get the cypress spec files as 'string', replace newline by ',' to match cypress --spec option
  cypressSpecFilesByIndex=$(tr '\n' ',' < "$indexFileName")

  echo "$cypressSpecFilesByIndex"
}
