# Shortcuts File Format Documentation
Credits:

Thanks to:
- u/Shoculad (Learned about iCloud API from them)
- Various people from r/Shortcuts and RoutineHub discord
- sebj for https://github.com/sebj/iOS-Shortcuts-Reference
- ActuallyZach (Magic Variables)
- marcusrbrown (Re-enabling shorcut importing on iOS 13/14)

(This guide is mainly intended for iOS 13/14, however there is still some stuff here that applies to iOS 15 and iOS 12-)

### How shortcuts are stored

On iOS 13, shortcuts are stored in /var/mobile/Library/Shortcuts/Shortcuts.realm. However, iOS 14 changes this to be in Shortcuts.sqlite. You don't need to worry about this at the moment though - what you do need to understand however is the .shortcut format. On iOS 12/13/14 devices, a shortcut that's stored on a device can be exported as an unsigned .shortcut file by using the Get My Shortcuts action (You can use a Save File action to save the output). iOS 15 is slightly different, however: you can't export shortcuts on device as unsigned shortcuts using Get My Shortcuts, only signed. You can, however, get a unsigned .shortcut from the iCloud API. Let's upload a shortcut to iCloud, and imagine our link is https://www.icloud.com/shortcuts/77dfe31578ac4f6fb084ebb418b34a49. Change /shortcuts/ to /shortcuts/api/records/ (https://www.icloud.com/shortcuts/api/records/77dfe31578ac4f6fb084ebb418b34a49). The value for fields.shortcut.value.downloadURL should be the URL for the unsigned .shortcut (Note: If you opened in Safari, change \/ to / in the URL). After getting the unsigned .shortcut, rename this to a plist and you should be able to easily open this in Xcode (or, use set name to rename to something.plist with Do Not Include File Extension on, and Get Text from that).

So, now lets go over the keys:

- WFWorkflowClientVersion - This is the number that represents the client version that the shortcut was shared on.
- WFWorkflowImportQuestions - The import questions for the shortcut
- WFWorkflowName - Name of the shortcut
- WFWorkflowMinimumClientVersion - The minimum client version the shortcut can be imported in.
- WFWorkflowMinimumClientVersionString - Same thing as WFWorkflowMinimumClientVersion but a string.
- WFWorkflowIcon - The Icon
- WFWorkflowInputContentItemClasses - What the shortcut accepts as input
- WFWorkflowTypes - Hard to explain, use sebj's documentation instead if you want this explained, but you likely won't touch this anyway
- WFWorkflowActions - this is the big one that matters. It's an array of the different shortcut actions.

### Actions

We're going to have a shortcut with a comment action that contains "Chocolate". Let’s take a look at a simple comment action:

```
<dict>
			<key>WFWorkflowActionIdentifier</key>
			<string>is.workflow.actions.comment</string>
			<key>WFWorkflowActionParameters</key>
			<dict>
				<key>WFCommentActionText</key>
				<string>Chocolate</string>
			</dict>
		</dict>
```

The value “is.workflow.actions.comment” in the WFWorkflowActionIdentifier indicates that it’s a comment action, and the Chocolate value in the WFCommentActionText key indicates that the Comment action contains the text "Chocolate".

Just a warning for magic variables: these contain an invisible character. Be careful not to accidentally remove it. Here’s an example of what it looks like, with the invisible character replaced with "(INVISIBLE CHARACTER)" (Thanks to ActuallyZach for sending me this and helping me figure out where it was!):

```
<dict>
            <key>WFWorkflowActionIdentifier</key>
            <string>is.workflow.actions.gettext</string>
            <key>WFWorkflowActionParameters</key>
            <dict>
                <key>WFTextActionText</key>
                <dict>
                    <key>Value</key>
                    <dict>
                        <key>attachmentsByRange</key>
                        <dict>
                            <key>{15, 1}</key>
                            <dict>
                                <key>Aggrandizements</key>
                                <array/>
                                <key>Type</key>
                                <string>ExtensionInput</string>
                            </dict>
                        </dict>
                        <key>string</key>
                        <string>Shortcut Input (INVISIBLE CHARACTER)</string>
                    </dict>
                    <key>WFSerializationType</key>
                    <string>WFTextTokenString</string>
                </dict>
            </dict>
        </dict>
```

### Importing

On iOS 12, you could just import .shortcuts from files no issue. However, this was disabled on iOS 13. It's still fairly easy to work around this though - you could have a shortcut that passes a .shortcut file into the Get Link to File action, and it should generate an iCloud link to this .shortcut in which you can import. Similar behavior on iOS 14. (Fun fact: While this is disabled, the code for it is still there: you can actually make a backup, edit a boolean in a plist, restore from said backup and you can import .shortcuts again. Apple actually accidentally enabled it again in iOS 14b1, though quickly realized their mistake and disabled it again.)

On iOS 15, however, Apple introduced shortcuts signing, as well as removed the special treatment for Get Link to File regarding .shortcut/.wflow files. This means that you need to sign a shortcut file if you want to import it on an iOS 15 device - and no, you can't do it on device. (Note: There was a way to bypass signing and import unsigned shortcuts files on iOS 15b1, but it was quickly patched.) You need either a Mac or an iOS 12/13/14 device to sign said shortcut file. (To sign on macOS - use the shortcuts CLI tool. To sign on an iOS 12/13/14 device, have a shortcut get the iOS 15 shortcut you want to sign, and use the Get Link to File action to upload to iCloud and sign it. The link to the signed shortcut file should be in fields.signedShortcut.value.downloadURL. However, you *could* set up a shortcuts signing server if you have a Mac if you really want to - but be aware that if someone uploads a malicious shortcut, Apple could ban it.

### iOS 15b1 shortcuts signing bypass

On iOS 15b1, Apple forgot to enforce signing to *.wflow files (the old file extension that Workflow used before it became shortcuts, basically the same as the unsigned .shortcut file format) This means you could easily rename a .shortcut to .wflow and import it. Apple patched this in 15b2.

### Re-enabling shortcut importing on iOS 13/14:

As I mentioned earlier, while shortcut importing is disabled in iOS 13/14, the code for it is still there. By changing the WFShortcutsFileSharingEnabled boolean to true in Shortcut's Preferences plist, you can re-enable file importing. It's /private/var/mobile/Library/Preferences/com.apple.siri.shortcuts.plist if you want to edit it on a jailbroken device, or you could modify /HomeDomain/Library/Preferences/com.apple.siri.shortcuts.plist in a backup.
