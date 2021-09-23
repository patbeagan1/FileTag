# FileTag
A utility for tagging files, and being able to look them up by their tags. Based on symlinks. 

## Why?

[Fie tagging is a great idea](https://www.nayuki.io/page/designing-better-file-organization-around-tags-not-hierarchies). It lets us represent metadata about files that can be awkward or impossible to express in a hierarchal way. I spent a while looking for a tagging solution that would work on MacOS. There are several high quality solutions out there for Windows, and even Linux, such as:
- https://tmsu.org
- https://www.tagsistant.net

However there does not seem to be much demand for a Mac tagging solution. I would guess that this is for a couple of reasons:
- Users are generally unfamiliar with the concept when applied to computers
- Users don't generally spend much time setting up file organization strategies
- Tagging is already supported in `Finder` - where most people will reach first when doing file manipulation. 

I don't think that Finder's implementation is ideal though, for a couple of reasons:
- It's not portable, so if you move to a different file format you may lose your tags. 
- It's not great for scripting either, since finder is the one doing tag management
- It's not open source

And so, `FileTag` was created to solve these problems

## How does it work? 

`FileTag` will hash the file it's given, and create a symlink in the `~/.index` folder that points back to the original file. The name of the symlink is the name of the filehash. 

From that point on, metadata can be linked back to the hash of the file. There is an implicit guarantee that if the symlink ever goes bad, you'll be able to reindex the file to the same hash and recover your metadata. This is portable, and does not take up much space on disk, because the actual contents of the file are stored elsewhere. The files could even be on external media. 

## How do I use it? 

1. Adding the tag "y" to the file "tagger.py" within the tagspace "TEST"
```
% ./tagger.py -s TEST tagas y tagger.py
symlink created for: /Users/pbeagan/Downloads/FileTag/tagger.py
symlink created for: /Users/pbeagan/.index/id/7f405d781e6f3ab1bfdb4596b7827341e36e5573a8ba51ecd9bf187f3223668e
```
2. Verifying that the "TEST" tagspace contains the "y" tag
```
% ./tagger.py -s TEST taglist
y
```
3. Finding a list of files within the "TEST" tagspace that are tagged with "y". See that the ID matches!
```
% ./tagger.py -s TEST match y
/Users/pbeagan/.index/id/7f405d781e6f3ab1bfdb4596b7827341e36e5573a8ba51ecd9bf187f3223668e
```
You can now browse to `~/.index/tags_TEST` to preview any of the files that were tagged in this way, regardless of which disk/ filepath they are stored on.

You will probably want to create an alias for your default tagspace - something like 
```
alias filetag="~/FileTag/tagger.py -s default"
```

## Could I have more details?

```
% ./tagger.py

    This util sets up an index of tagged files at ~/.index/id.
    It uses symlinks based on the sha256 hash of an object to make sure the tags are accurate.

    tagger.py [-s TAGSPACE] SUBCMD X [...]
    Supported subcommands:

        help|-h
            Print this usage guide
        tagcheck
            Print the tags that are associated with the following list of files
        tagas
            Tag as X the following list of files
        tag
            Tag X file with the following list of tags
        taglist
            List out the available tags in the current tagspace
        index
            Reindex the file X
        base
            Prints the base file that the tag symlink would eventually point to
        match
            Print files that have all of the following tags
            match -o will open all matched files
        matchany
            Print files that have any of the following tags
        meta
            add - add metadata to a metadata file
            ls - look at metadata for a file
```

### Can you share some useful one liners? 

Check for files that are tagged with "mytag", with in the "default" tagspace. Then, report on whether those files are available (unbroken symlinks)
```sh
./tagger.py -s default match <mytag> | xargs ./tagger.py base
```
