1. Open Terminal in your Project Folder

cd "/Users/bilsonyumnam/Desktop/fullstack project/CMS"

2. Remove any old notes folder (if it already exists)
bash

rm -rf ./notes

3. Create the Link to your NOTES-cms- vault
bash
ln -s ~/NOTES-cms- ./notes
(If your NOTES-cms- is located somewhere else, replace ~/NOTES-cms- with the exact path).

./notes is the notes folder you created at the root of the repo you are working on.( if not created after step 2 create one )

4. Verify it worked
Run ls ./notes — you should immediately see your How to structure note folder! 