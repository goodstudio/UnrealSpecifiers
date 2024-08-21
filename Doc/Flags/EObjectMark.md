|Name                                |Value               |Description                                                                                                                                                                                                                                                     |
|------------------------------------|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|OBJECTMARK_NOMARK                   |0x00000000          |Zero, nothing                                                                                                                                                                                                                                                   |
|OBJECTMARK_Saved                    |0x00000004          |Object has been saved via SavePackage. Temporary.                                                                                                                                                                                                               |
|OBJECTMARK_TagImp                   |0x00000008          |Temporary import tag in load/save.                                                                                                                                                                                                                              |
|OBJECTMARK_TagExp                   |0x00000010          |Temporary export tag in load/save.                                                                                                                                                                                                                              |
|OBJECTMARK_NotForClient             |0x00000020          |Temporary save tag for client load flag.                                                                                                                                                                                                                        |
|OBJECTMARK_NotForServer             |0x00000040          |Temporary save tag for server load flag.                                                                                                                                                                                                                        |
|OBJECTMARK_NotAlwaysLoadedForEditorGame|0x00000080          |Temporary save tag for editorgame load flag.                                                                                                                                                                                                                    |
|OBJECTMARK_EditorOnly               |0x00000100          |Temporary editor only flag                                                                                                                                                                                                                                      |
|OBJECTMARK_NotForTargetPlatform     |0x00000200          |Temporary save tag for stripping objets based on TargetPlatform                                                                                                                                                                                                 |