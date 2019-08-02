# TODO
- clarify usages of terms `cache` and `remote`

# DVC dependency management

This post is a follow up to [A walkthrough of DVC](https://blog.codecentric.de/en/2019/03/walkthrough-dvc/) and deals with managing dependencies between DVC projects.
In particular, this follow up is about importing specific versions of a model from one DVC project into another.

![pipeline](images/logo-owl-readme.png)

After a quick recap of the original walkthrough, in the first part of this article we created an example project, which you can use as a DVC-dependendy hands-on in the second part.
(Note that, for those of you in a hurry, part one can be skipped.
A publicly accessible example project is provided, such that you can step right into the hands-on fun in part two.)

## Recap of the original walkthrough
In [A walkthrough of DVC](https://blog.codecentric.de/en/2019/03/walkthrough-dvc/) we trained a model to classify hand-written numbers.
The walkthrough showed how implementing a DVC-pipeline makes all of data loading, preprocessing, training, performance evaluation, etc. fully reproducible.
The gist is that DVC _versions_ training data, (hyper-)parameters, code and trained models _together_.
In particular, a DVC project builds on top of a git repository, which implements all necessary versioning.

You might want to browse through the walkthrough first, such that you can get the most out of this post.
For those of you in a hurry, here is a quick summary of the walkthrough.
We create an ML pipeline to classify hand-written numbers.
The definition of the pipeline is versioned in the git repository in which the DVC project resides (see the following figure).
Binary data, such as e.g. training data and trained models, are located in DVC's so-called cache.
In particular, for each version of the pipeline, the cache contains different versions of all binary data.
However, cache data is _not_ stored in the git repository itself, but in a so-called remote, which resides outside of the git repository.
When checking out a specific version of the pipeline from the git repository, DVC takes care of fetching cache data from the remote that matches the current pipeline version.

![dvc remote](https://blog.codecentric.de/files/2019/03/dvc_remote.jpg)

## Creating the playground

As for the original walkthrough, [the GitHub repository](https://github.com/bbesser/dvc-deps-management) for the post you are reading now provides a readily usable working environment.
In this environment, you can interactively create the number classifier project (or let a script perform all actions for you).
Compared to the original walkthrough, the following extensions were implemented.
First, the created DVC project is pushed to GitHub such that it can easily be referenced as a DVC-dependency.
Secondly, the project's DVC cache is now pushed to a remote in an Amazon S3 bucket, such that binary data (e.g. trained models) can also be accessed from the internet.

[The GitHub repository](https://github.com/bbesser/dvc-deps-management) contains a readily usable working environment in which you can execute the extended walkthrough.
To prepare the working environment (see the following code block), clone [the GitHub repository](https://github.com/bbesser/dvc-deps-management), change into the cloned directory, and start the working environment using `./start_environment.sh bash`.
You will be 'logged in' to a newly created container.
From the prompt in the container, onfigure variables at the top of the file `/home/dvc/scripts/walkthrough.sh` to match your GitHub repository and S3 bucket, where both must be empty and writable.
(Details of the bucket configuration are discussed in section [Configuring the S3 remote](#s3remote).)
Then, you might want to run `/home/dvc/scripts/walkthrough.sh` to automatically perform all steps from the extended walkthrough.
After this is finished, the GitHub repository and the S3 bucket will now contain the DVC project and its cache, respectively.

<pre>
# $ is the host prompt in the cloned folder
# $$ is the container prompt in the working folder /home/dvc/walkthrough

$ git clone https://github.com/bbesser/dvc-deps-management
$ cd dvc-deps-management
$ vi scripts/walkthrough.sh # configure GitHub and S3
$ ./start_environment.sh bash
$$ /home/dvc/scripts/walkthrough.sh # creates DVC project and its cache
</pre>

### <a name="s3remote"></a>Configuring the S3 remote
In this section, we take a quick look at the part of the `walkthrough.sh` script that sets up an S3 bucket as the DVC-cache's remote.

In order to enable DVC to access a bucket, two preparations have to be done.
1. First, the `boto3` library must be installed, using `pip install boto3`. (This is already done in the working environment provided with this article.)
1. Secondly, `boto3` must be given access to the bucket.
Therefore, AWS credentials can be provided as environment variables like in the following code block.
(For the provided working environment, this configuration has to be done at the top of the `walkthrough.sh` script.)
```
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
# from here on, DVC can interact with your bucket 
``` 
Other means of configuring S3 bucket access for DVC/`boto3` are [documented here](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/configuration.html).

Adding an S3 bucket as remote to a DVC project is the same as adding any other type of remote (see the following code block or `scripts/walkthrough.sh`).
From the given URI, DVC knows that the remote should reside in a bucket.
The `-d` flag tells DVC that this remote should be used by default.
Once the bucket is added, the DVC pipeline's configuration in `.dvc/config` should be saved, by committing the changes to git.

```
dvc remote add -d the_remote s3://YOUR_BUCKET_NAME
git add .dvc/config # save remote configuration, such that cached data can be pulled from your new remote when your team colleagues checkout the git repo
```

## Using a DVC project as a dependency
If you're not having the time to execute the extended walkthrough, you can step into hands-on fun from here.
[This publicly available DVC project](https://github.com/bbesser/dvc-deps-management-companion) was created by the extended walkthrough and can be used as a DVC dependency.
The S3 remote for the project's DVC cache is publicly readable.

### dvc get and dvc import

We discuss how to access a DVC project's _outputs_.
An output is some (possibly binary) file created by the pipeline defined in the project.
To be precise, an output file is created by some stage of the pipeline, e.g. the training stage creates a trained model as output.
Recall that the pipeline and its output file are versioned.
Consequently, any version of any output file can be accessed.

We present two ways to access an output, namely `dvc get` and `dvc import`.
Intuitively, `dvc get` simply 'downloads' an output file.
`dvc import` downloads an output file and additionally considers what version was fetched.

TODO (package management, ci) 

### dvc get

TODO


# Notes
now you can try `dvc get` as follows
<pre>
virtualenv tmpvenv
source tmpvenv/bin/activate
pip install git+git://github.com/iterative/dvc@0.52.1
pip install boto3 # for s3 access
# these credentials allow read access to project's s3 bucket only
export AWS_ACCESS_KEY_ID=AKIAUTWTWR37V2Q623BI
export AWS_SECRET_ACCESS_KEY=EyZt43a84c [Compare versions](#compare-versions)YzAngl/mLRwlCU7YAUB6iMInFhlc1M
dvc get --rev 0.3 https://github.com/bbesser/dvc-deps-management-companion.git model/model.h5
ls model.h5
</pre>

Im Artikel wird nur `dvc import` beleuchtet, da es die allgemeinere Variante von `dvc get` ist. `dvc get` wir dann im Abschluss erwaehnt.

# End - old post from here on

## Prepare the repository
The working folder `/home/dvc/walkthrough` already contains the subfolder `code` holding the required code. Let us turn `/home/dvc/walkthrough` into a "DVC-enabled" Git repository. DVC is built on top of Git. All DVC configuration is versioned in the same Git repository as your model code, in the subfolder `.dvc`, see the following code block. Note that tagging this freshly initialized repository is not a must&mdash;we create a tag only for the purpose of later parts of this walkthrough.

<pre>
# Code is shortened, for brevity.
# See /home/dvc/scripts/walkthrough.sh for complete code.

$$ git init
$$ git add code
$$ git commit -m "initial import"
$$ dvc init
$$ git status
        new file: .dvc/.gitignore
        new file: .dvc/config
$$ git add .dvc
$$ git commit -m "init dvc"
$$ git tag -a 0.0 -m "freshly initialized with no pipeline defined, yet"
</pre>

## Define the pipeline
Our pipeline consists of three stages, namely
1. loading data,
2. training, and
3. evalutation,

and the pipeline also produces performance metrics of the trained model. Here is a schematic:

![pipeline](https://blog.codecentric.de/files/2019/03/pipeline-3.jpg)

For simplicity, we implement a dummy loading stage, which just copies given raw image data into the repository. However, since our goal is to retrain the model as more and more data is available, our loading stage can be configured for the amount of data to be copied. This configuration is located in the file `config/load.json`. Similarly, training stage configuration is located in `config/train.json` (our neural network's architecture allows to alter the number of convolution filters). Let's put this congiuration under version control.

<pre>
$$ mkdir config
$$ echo '{ "num_images" : 1000 }' > config/load.json
$$ echo '{ "num_conv_filters" : 32 }' > config/train.json
$$ git add config/load.json config/train.json
$$ git commit -m "add config"
</pre>

Stages of a DVC pipeline are connected by *dependencies* and *outputs*. Dependencies and outputs are simply files. E.g. our load stage *depends* on the configuration file `config/load.json`. The load stage *outputs* training image data. If upon execution of our load stage the set of training images changes, the training stage picks up these changes, since it depends on the training images. Similarly, a changed model will be evaluated to obtain its performance metrics. Once the pipeline definition is in place, DVC takes care of reproducing only those stages with changed dependencies, as we discuss in detail in section [Reproduce the pipeline](#reproduce-the-pipeline).

The following `dvc run` command configures our load stage, where the stage's definition is stored in the file given by the `-f` parameter, dependencies are provided using the `-d` parameter, and training image data is output into the folder `data` given by the `-o` parameter. DVC immediately executes the stage, already generating the desired training data.

<pre>
$$ dvc run -f load.dvc -d config/load.json -o data python code/load.py
Running command:
        python code/load.py
Computing md5 for a large directory data/2. This is only done once.
...
$$ git status
        .gitignore
        load.dvc
$$ cat .gitignore
data
$$ git add .gitignore load.dvc
git commit -m "init load stage"
</pre>

Recall that our load stage outputs image data into the folder `data`. DVC has added the `data` folder to Git's ignore list. This is because large binary files are not to be versioned in Git repositories. See section [DVC-cached files](#dvc-cached-files) on how DVC manages such data.

After adding `.gitignore` and `load.dvc` to version control, we define the other two stages of our pipeline analogously, see the following code block. Note the dependency of our training stage to the training config file. Since training typically takes long times (not so in our toy project, though), we output the readily trained model into a file, namely `model/model.h5`. As DVC versions this binary file, we have easy access to this version of our model in the future.

<pre>
$$ dvc run -f train.dvc -d data -d config/train.json -o model/model.h5 python code/train.py
...
$$ dvc run -f evaluate.dvc -d model/model.h5 -M model/metrics.json python code/evaluate.py
...
</pre>

For the evaluation stage, observe the definition of the file `model/metrics.json` as a *metric* (`-M` parameter). Metrics can be inspected using the `dvc metrics` command, as we discuss in section [Compare versions](#compare-versions). To wrap up our first version of the pipeline, we put all stage definitions (`.dvc`-files) under version control and add a tag.

<pre>
$$ git add ...
$$ git commit ...
$$ git tag -a 0.1 -m "initial pipeline version 0.1"
</pre>

*Remark*: DVC does not only support tags for organizing versions of your pipeline, it also allows to utilize branch structures.

Finally, let's have a brief look at how DVC renders our pipeline.

<pre>
$$ dvc pipeline show --ascii evaluate.dvc
</pre>

![pipeline rendered by dvc](https://blog.codecentric.de/files/2019/03/pipeline-dvc-1.jpg)

*Remark*: Observe that stage definitions call arbitrary commands, i.e., DVC is language-agnostic and not bound to Python. No one can stop you from implementing stages in Bash, C, or any other of your favorite languages and frameworks like R, Spark, PyTorch, etc.

## <a name="dvc-cached-files"></a>DVC-cached files
For building up intuition on how DVC and Git work together, let us skip back to our initial Git repository version. Since no pipeline is defined, yet, none of our training data, model, or metrics exist.

Recall that DVC uses Git to keep track of which output data belongs to the checked out version. Therefore, *additionally* to choosing the version via the `git` command, we have to instruct DVC to synchronize outputs using the `dvc checkout` command. I.e., as when initializing the repository `git` and `dvc` have to be used in tandem.

<pre>
$$ git checkout 0.0
$$ dvc checkout
$$ ls data
ls: cannot access 'data': No such file or directory # desired :-)
</pre>

Back to the latest version, we find that DVC has restored all training data.

<pre>
$$ git checkout 0.1
$$ dvc checkout
$$ ls data
0  1  2  3  4  5  6  7  8  9 # one folder of images for each digit
</pre>

Similarly, you can skip to any of your versioned pipelines and inspect their configuration, training data, models, metrics, etc.

*Remark*:  Recall that DVC configures Git to ignore output data. How is versioning of such data implemented? DVC manages output data in the repository's subfolder `.dvc/cache` (which is also ignored by Git, as configured in `.dvc/.gitignore`). DVC-cached files are exposed to us as hardlinks from output files into DVC's cache folder, where DVC takes care of managing the hardlinks.

![dvc cache](https://blog.codecentric.de/files/2019/03/dvc_cache.jpg)

## <a name="reproduce-the-pipeline"></a>Reproduce the pipeline
Pat yourself on the back. You have mastered *building* a pipeline, which is the hard part. *Reproducing* (parts of) it, i.e., re-executing stages with changed dependencies, is super-easy. First, note that if we do not change any dependencies, there is nothing to be reproduced.

<pre>
$$ dvc repro evaluate.dvc
...
Stage 'load.dvc' didnt change.
Stage 'train.dvc' didnt change.
Stage 'evaluate.dvc' didnt change.
Pipeline is up to date. Nothing to reproduce.
</pre>

When changing the amount of training data (see pen icon in the following figure) and calling the `dvc repro`-command with parameter `evaluate.dvc` for the last stage (red play icon), the entire pipeline will be reproduced (red arrows).

![reproduce-all](https://blog.codecentric.de/files/2019/03/pipeline-repro-all-1.jpg)

<pre>
$$ echo '{ "num_images" : 2000 }' > config/load.json
$$ dvc repro evaluate.dvc
...
Warning: Dependency 'config/load.json' of 'load.dvc' changed.
Stage 'load.dvc' changed.
Reproducing 'load.dvc'
...
Warning: Dependency 'data' of 'train.dvc' changed.
Stage 'train.dvc' changed.
Reproducing 'train.dvc'
...
Warning: Dependency 'model/model.h5' of 'evaluate.dvc' changed.
Stage 'evaluate.dvc' changed.
Reproducing 'evaluate.dvc'
</pre>

Observe that DVC tracks changes to dependencies and outputs through md5-sums stored in their corresponding stage's `.dvc`-file:

<pre>
$$ git status
        modified:   config/load.json
        modified:   evaluate.dvc
        modified:   model/metrics.json
        modified:   load.dvc
        modified:   train.dvc
$$ git diff load.dvc
...
deps:
-- md5: 44260f0cf26e82df91b23ab9a75bf4ae
+- md5: 2e13ea50a381a9be4809026b71210d5a
...
outs:
-  md5: b5136af809e828b0c51735f40c5d6db6.dir
+  md5: 8265c35e6eb17e79de9705cbbbd9a515.dir
</pre>

Let us save this version of our pipeline and tag it.

<pre>
$$ git add load.dvc train.dvc evaluate.dvc config/load.json model/metrics.json
$$ git commit -m "0.2 more training data"
$$ git tag -a 0.2 -m "0.2 more training data"
</pre>

## Reproduce partially
What if only training *configuration* changes, but training *data* remains the same? All stages but the load stage should be reproduced. We have control over which stages of the pipeline are reproduced. In a first step, we reproduce only the training stage by issuing the `dvc repro` command with parameter `train.dvc`, the stage in the middle of the pipeline (we increase the number of convolution filters in our neural network).

![reproduce-all](https://blog.codecentric.de/files/2019/03/pipeline-repro-train-1.jpg)

<pre>
$$ echo '{ "num_conv_filters" : 64 }' > config/train.json
$$ dvc repro train.dvc
...
Stage 'load.dvc' didnt change.
Warning: Dependency 'config/train.json' of 'train.dvc' changed.
Stage 'train.dvc' changed.
Reproducing 'train.dvc'...
</pre>

We can now reproduce the entire pipeline. Since we already performed re-training, the trained model was changed also, and only the evaluation stage will be executed.

![reproduce-all](https://blog.codecentric.de/files/2019/03/pipeline-repro-evaluate-1.jpg)

<pre>
$$ dvc repro evaluate.dvc
...
Stage 'load.dvc' didnt change.
Stage 'train.dvc' didnt change.
Warning: Dependency 'model/model.h5' of 'evaluate.dvc' changed.
Stage 'evaluate.dvc' changed.
Reproducing 'evaluate.dvc'...
</pre>

Finally, let us increase the amount of available training data and trigger the entire pipeline by reproducing the `evaluate.dvc` stage.

<pre>
$$ echo '{ "num_images" : 3000 }' > config/load.json
$$ dvc repro evaluate.dvc
...
Warning: Dependency 'config/load.json' of 'load.dvc' changed.
Stage 'load.dvc' changed.
Reproducing 'load.dvc'
...
Warning: Dependency 'data' of 'train.dvc' changed.
Stage 'train.dvc' changed.
Reproducing 'train.dvc'
...
Warning: Dependency 'model/model.h5' of 'evaluate.dvc' changed.
Stage 'evaluate.dvc' changed.
Reproducing 'evaluate.dvc'...
</pre>

Again, we save this version of our pipeline and tag it.

<pre>
$$ git add config/load.json config/train.json evaluate.dvc load.dvc train.dvc model/metrics.json
$$ git commit -m "0.3 more training data, more convolutions"
$$ git tag -a 0.3 -m "0.3 more training data, more convolutions"
</pre>

## <a name="compare-versions"></a>Compare versions
Recall that we have defined a *metric* for the evaluation stage, in the file `model/metrics.json`. DVC can list metrics files for all tags in the entire Git repository, which allows us to compare model performances for various all versions of our pipeline. Clearly, increasing the amount of training data and adding convolution filters to the neural network improves the model's accuracy.

<pre>
$$ dvc metrics show -T # -T for all tags
...
0.1:
        model/metrics.json: [0.896969696969697]
0.2:
        model/metrics.json: [0.9196969693357294]
0.3:
        model/metrics.json: [0.9565656557227626]
</pre>

Actually, the file `model/metrics.json` stores not only the model's accuracy, but also its loss. To display only the accuracy, we have configured DVC with an XPath expression as follows. This expression is stored in the corresponding stage's `.dvc` file.

<pre>
$$ dvc metrics modify model/metrics.json --type json --xpath acc
$$ cat evaluate.dvc
...
metric:
 type: json
 xpath: acc
...
</pre>

*Remark 1*: DVC also supports metrics stored in CSV files or plain text files. In particular, DVC does not interpret metrics, and instead treats them as plain text.

*Remark 2*: For consistent metrics display over all pipeline versions, metrics should be configured in the very beginning of your project. In this case, configuration contained in `.dvc`-files is the same for all versions.

## Share data
When developing models in teams, sharing training data, readily trained models, and performance metrics is crucial for efficient collaboration--each team member retraining the same model is a waste of time. Recall that stage output data is not stored in the Git repository. Instead, DVC manages these files in its `.dvc/cache` folder. DVC allows to push cached files to remote storage (SSH, NAS, Amazon, S3, ...). From there, each team member can pull that data to their individual workspace's DVC cache and work with it as usual.

![dvc remote](https://blog.codecentric.de/files/2019/03/dvc_remote.jpg)

For the purpose of this walkthrough, we fake remote storage using a local folder called `/remote`. Here is how to configure the remote and push data to it.

<pre>
$$ mkdir /remote/dvc-cache
$$ dvc remote add -d fake_remote /remote/dvc-cache # -d for making the remote default
$$ git add .dvc/config # save the remote's configuration
$$ git commit -m "configure remote"
$$ dvc push -T
</pre>

The `-T` parameter pushes cached files for all tags. Note that `dvc push` intelligently pushes only new or changed data, and skips over data that has remained the same since the last push.

How would your team member access your pushed data? (If you followed along in your shell, exit the container and recreate it by calling `./start_environment.sh bash`. The following steps are documented in `/home/dvc/scripts/clone.sh` and should be applied in the `/home/dvc`-folder.) Recall that cloning the Git repository will *not* checkout training data, etc. since such files are managed in by DVC. We need to instruct DVC to pull that data from the remote storage. Thereafter, we can access the data as before.

<pre>
$$ cd /home/dvc
$$ git clone /remote/git-repo walkthrough-cloned
$$ cd walkthrough-cloned
$$ ls data
ls: cannot access 'data': No such file or directory # no training data there :(
$$ dvc pull -T # -T to pull for all tags
$$ ls data
0  1  2  3  4  5  6  7  8  9 # theeere is our training data :)
</pre>

*Remark*: Pushing/pulling DVC-managed data for all tags (`-T` parameter) is not advisable in general, since you will send/receive *lots* of data.

## Conclusion
DVC allows you to define (language-agnostic) reproducible ML pipelines and version pipelines *together with* their associated training data, configuration, performance metrics, etc. Performance metrics can be evaluated for all versions of a pipeline. Training data, trained models, and other associated binary data can be shared (storage-agnostic) with team members for efficient collaboration.
