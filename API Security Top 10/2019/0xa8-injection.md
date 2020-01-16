API8:2019 Injection (�C���W�F�N�V����)
===================

| ���ЃG�[�W�F���g/�U���o�H | �Z�L�����e�B��̎�_ | �e�� |
| - | - | - |
| API �ˑ� : �U����Փx **3** | �����x **2** : ���o��Փx **3** | �Z�p�I�e�� **3** : �r�W�l�X�ˑ� |
| �U���҂́A���p�\�Ȃ�����C���W�F�N�V�����x�N�^ (���Ƃ��΁A���ړ��́A�p�����[�^�A�����T�[�r�X�Ȃ�) ����Ĉ��ӂ���f�[�^�� API �ɋ������A�C���^�v���^�ɑ��M����邱�Ƃ����҂���B | �C���W�F�N�V�����̌��ׂ́A���Ɉ�ʓI�ł���ASQL�ALDAP�ANoSQL �̃N�G���� OS �R�}���h�AXML �p�[�T�AORM �ł悭������B�����̌��ׂ́A�\�[�X�R�[�h���r���[���ɗe�ՂɌ��o�ł���B�U���҂̓X�L���i��t�@�U�[���g�p����\��������B | �C���W�F�N�V�����́A���R������f�[�^�����ɂȂ���\��������B�܂��ADoS �⊮�S�ȃz�X�g�̏�����ɂȂ���\��������B |

## API ���Ǝォ�ǂ����̊m�F

�ȉ��̏ꍇ�AAPI �̓C���W�F�N�V�����̌��ׂɑ΂��ĐƎ�ł���B

* �N���C�A���g����񋟂����f�[�^���AAPI �ɂ���Č��؁A�t�B���^�����O�A�T�j�^�C�W���O����Ă��Ȃ��B
* Client-supplied data is directly used or concatenated to SQL/NoSQL/LDAP queries, OS commands, XML parsers, and Object Relational Mapping (ORM)/Object Document Mapper (ODM).
�N���C�A���g����񋟂����f�[�^���ASQL/NoSQL/LDAP �N�G���� OS �R�}���h�AXML �p�[�T�A�I�u�W�F�N�g�֌W�}�b�s���O (ORM) �ɒ��ڎg�p�E�A������Ă���B
* �O���V�X�e�� (���Ƃ��΁A�����V�X�e��) ��������Ă���f�[�^���AAPI �ɂ���Č��؁A�t�B���^�����O�A�T�j�^�C�W���O����Ă��Ȃ��B

## �U���V�i���I��

### �V�i���I #1

�y�A�����^���E�R���g���[���f�o�C�X�̃t�@�[���E�F�A�́AappId ���}���`�p�[�g�p�����[�^�Ƃ��đ��M����邱�Ƃ�z�肵�Ă���G���h�|�C���g `/api/CONFIG/restore` ��񋟂���B�f�R���p�C����p���邱�ƂŁA�U���҂́AappId ���T�j�^�C�W���O�Ȃ��ŃV�X�e���R�[���ɒ��ړn����邱�Ƃ𔭌�����B

```c
snprintf(cmd, 128, "%srestore_backup.sh /tmp/postfile.bin %s %d",
         "/mnt/shares/usr/bin/scripts/", appid, 66);
system(cmd);
```

�ȉ��̃R�}���h�ŁA�U���҂͓��l�̐Ǝ�ȃt�@�[���E�F�A�����C�ӂ̃f�o�C�X���V���b�g�_�E�����邱�Ƃ��ł���B

```
$ curl -k "https://${deviceIP}:4567/api/CONFIG/restore" -F 'appid=$(/etc/pod/power_down.sh)'
```

### �V�i���I #2

�\�񂪂���I�y���[�V�����̂��߂̊�{�I�� CRUD �@�\�����A�v���P�[�V���������݂���B�U���҂́A�\��폜�̃��N�G�X�g�� `bookingId` �N�G���X�g�����O�p�����[�^��p���āANoSQL �C���W�F�N�V�������\�ł��邱�Ƃ����Ƃ����肵���B����́A���N�G�X�g�� `DELETE /api/bookings?bookingId=678` �̂悤�ɂȂ��Ă��邾�낤�BAPI �T�[�o�͈ȉ��̊֐���p���āA�폜���N�G�X�g����������B

```javascript
router.delete('/bookings', async function (req, res, next) {
  try {
      const deletedBooking = await Bookings.findOneAndRemove({'_id' : req.query.bookingId});
      res.status(200);
  } catch (err) {
     res.status(400).json({error: 'Unexpected error occured while processing a request'});
  }
});
```

�U���҂́A���N�G�X�g��T�󂵁A`bookingId` �N�G���X�g�����O�p�����[�^���ȉ��Ɏ����悤�ɕύX����B���̃P�[�X�ł́A�U���҂͑����[�U�̗\����폜���邱�Ƃ��ł���B

```
DELETE /api/bookings?bookingId[$ne]=678
```

## �΍����@

�C���W�F�N�V������h�����߂ɂ́A�R�}���h�ƃN�G������f�[�^�𕪂���K�v������B

* �P��́A�M���ł���A�����Ɉێ�����Ă��郉�C�u������p���ăf�[�^���؂��s���B
* �N���C�A���g����񋟂���r���S�Ẵf�[�^�ⓝ���V�X�e�����痈�鑼�̃f�[�^�̌��؁A�t�B���^�����O�A�T�j�^�C�W���O���s���B
* ���ꕶ���́A�Ώۂ̃C���^�v���^�̓��L�̃V���^�b�N�X��p���ăG�X�P�[�v�����ׂ��ł���B
* �p�����[�^�����ꂽ�C���^�t�F�C�X��񋟂�����S�� API ��I������B
* �Ԃ��Ă��郌�R�[�h�̐�����ɐ������A�C���W�F�N�V�������N�������ۂ̑�ʂ̘R������h�~����B
* �\���ȃt�B���^��p���ē��̓f�[�^�����؂��A�e���̓p�����[�^�ɗL���Ȓl�̏ꍇ�̂݋�����B
* �S�Ă̕�����p�����[�^�ɑ΂��ăf�[�^�^�C�v�ƌ����ȃp�^�[�����`����B

## References

### OWASP

* [OWASP Injection Flaws][1]
* [SQL Injection][2]
* [NoSQL Injection Fun with Objects and Arrays][3]
* [Command Injection][4]

### External

* [CWE-77: Command Injection][5]
* [CWE-89: SQL Injection][6]

[1]: https://www.owasp.org/index.php/Injection_Flaws
[2]: https://www.owasp.org/index.php/SQL_Injection
[3]: https://www.owasp.org/images/e/ed/GOD16-NOSQL.pdf
[4]: https://www.owasp.org/index.php/Command_Injection
[5]: https://cwe.mitre.org/data/definitions/77.html
[6]: https://cwe.mitre.org/data/definitions/89.html
