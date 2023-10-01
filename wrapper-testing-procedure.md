# Wrapper Testing Procedure
The following is a standard procedue for testing service wrappers. This does not include testing the upstream service, which is different for every service and must be outlined by the service owner.

*During testing, put yourself in the shoes of a user who knows little to nothing about StartOS or the service.*

### Marketplace list
- good icon, accurate title, and a clear short description

### Marketplace show
- helpful release notes
- view "All Release Notes" to make sure there are no errors and iold release notes (if any) display as expected
- clear long description
- make sure dependencies look accurate (especially "required" vs "required by default" vs "optional")
- click all buttons under "Additional Info" to make sure they work
- read the instructions as if you were a five-year-old
- install the service

### Installed Show
- determine whether or not the service properly entered (or did not enter) "Needs Config" state
- explore the service config (even if unnecessary). Change things, make sure everything happens as you would expect. Save if necessary
- start the service
- make sure health checks to appear as expected. They should have helpful titles and progress from "Waiting for response" to "Starting" to "Success" as quickly as possible. Also beside these statuses, there should be helpful messaging in most cases, especially if the title is not fully self-explanitory
- view the service Logs (be sure to scroll back all the way to the beginning). Look for anything weird, especially errors
- view the service Properties. Make sure all properties display as you would expect
- view the service Actions. Perform each action.
- view the service Interfaces. Look for anything weird.
- click the donate button. It should either inform you the service does not accept donations or take you to the correct donation page
- finally, use the service as described in Instructions. If the service has a UI, launch it. If the service is meant to be used from another client, go get that client. You do not have to perform comprehensive testing of the service itself, just genreally follow the instructions and make sure nothing obvious is broken
- restart the service
- view logs during restart. Be sure to scroll back and view the logs that were generated during testing. As before, you looking for anything weird, especially erros

### Backups
- make a backup of the service
- uninstall the service
- restore the service from backup
