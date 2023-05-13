```bash
index=main sourcetype="GitLab:Commit" 
| sort - _time 
| table _time id message diff_output{}.deleted_file diff_output{}.new_file diff_output{}.new_path diff_output{}.old_path diff_output{}.renamed_file 
| rename diff_output{}.* as * 
| eval ziparray=mvzip(mvzip(mvzip(mvzip(deleted_file,new_file,";"),new_path,";"),old_path,";"),renamed_file,";") 
| mvexpand ziparray 
| fields id message ziparray 
| eval ziparray=split(ziparray,";"), deleted_file=mvindex(ziparray,0), new_file=mvindex(ziparray,1), new_path=mvindex(ziparray,2), old_path=mvindex(ziparray,3), renamed_file=mvindex(ziparray,4) 
| fields - ziparray 
| eval commit_changes = case(deleted_file=="true","File ".old_path." is deleted",new_file=="true","File ".new_path." is created", renamed_file=="true","File ".old_path." renamed to ".new_path,deleted_file=="false" AND new_file=="false" AND renamed_file="false","File ".new_path." is updated",1=1,"null") 
| fields _time id message commit_changes 
| stats values(commit_changes) as commit_changes by id, message
```
