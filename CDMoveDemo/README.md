#  Project notes

## Notes on `replacePersistentStore(...)`

- This could almost be a class method, apparently, because it doesn't matter if the receiver has loaded either of the persistent stores. Backing up seems to work just as well with a plain `NSPersistentStoreCoordinator()` as the target.
    - If you do that you'll get this in the console, but it works anyway if the destination isn't used by a currently loaded store:
    
            2021-03-22 16:49:49.656252-0600 CDMoveDemo[12693:5017658] [error] warning: client failed to call designated initializer on NSPersistentStoreCoordinator
            CoreData: warning: client failed to call designated initializer on NSPersistentStoreCoordinator
            
    - Attempting to restore with this fails. The `replace` call appears to work (it doesn't throw, and it doesn't return any status) but there's no apparent effect
    - Initializing a PSC with the current container's data model quiets the warning but has no apparent effect on how `replacePersistentStore` works.
- If the destination URL refers to a currently loaded store, that store is removed from the PSC. You'll need to re-add it after the call. See the code for restoring from a backup.
- From [a dev forum post in summer 2020](https://developer.apple.com/forums/thread/651325):

    > Additionally you should almost never use NSPersistentStoreCoordinator's migratePersistentStore... method but instead use the newer replacePersistentStoreAtURL.. (you can replace emptiness to make a copy). The former loads the store into memory so you can do fairly radical things like write it out as a different store type. It pre-dates iOS. The latter will perform an APFS clone where possible.
    
- From WWDC 2015, ["What's New in Core Data"](https://developer.apple.com/videos/play/wwdc2015/220/), `replacePersistentStore`
    - Has the same pattern as `destroyPersistentStoreAtURL` (from earlier in that session)
    - If destination doesn’t exist, this does a copy
    - As for `destroy...`, the previous slide notes that it
        - Honors locking protocols
        - Handles details reconfiguring emptied files
        - Journal mode, page size, etc.
        - Need to pass same options as addToPersistentStore 
        - Accidentally switching journal modes can deadlock
- From [WWDC 2016](https://developer.apple.com/videos/play/wwdc2016/242/),
    - `replace...` is safe to use regardless of whether the store is open
- Some notes from `NSPersistentStoreCoordinator.h`:

    First, it's possible to force an "unsafe" operation.

        /* store option for the destroy... and replace... to indicate that the store file should be destroyed even if the operation might be unsafe (overriding locks
        */
        COREDATA_EXTERN NSString * const NSPersistentStoreForceDestroyOption API_AVAILABLE(macosx(10.8),ios(6.0));

    Next, the `replace...` method has this note

        /* copy or overwrite the target persistent store in accordance with the store class's requirements.  It is important to pass similar options as addPersistentStoreWithType: ... SQLite stores will honor file locks, journal files, journaling modes, and other intricacies.  Other stores will default to using NSFileManager.
        */
