#  CDMoveDemo

This app demonstrates backing up and restoring Core Data persistent stores.

The UI is just a table view that lists trivial managed object entries, sorted by their timestamp field. The buttons at the top are as follows:

- `+`: Add a new entry
- Down arrow: Back up the current persistent store
- Up arrow: Restore the previously backed up persistent store.

The guts of backing up and restoring are found in an `NSPersistentContainer+extension.swift`. The process is described in detail in a blog post at <https://atomicbird.com/blog/core-data-back-up-store/>.

