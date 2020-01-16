API7:2019 Security Misconfiguration (�Z�L�����e�B�ݒ�~�X)
===================================

| ���ЃG�[�W�F���g/�U���o�H | �Z�L�����e�B��̎�_ | �e�� |
| - | - | - |
| API �ˑ� : �U����Փx **3** | �����x **3** : ���o��Փx **3** | �Z�p�I�e�� **2** : �r�W�l�X�ˑ� |
| Attackers will often attempt to find unpatched flaws, common endpoints, or unprotected files and directories to gain unauthorized access or knowledge of the system. | Security misconfiguration can happen at any level of the API stack, from the network level to the application level. Automated tools are available to detect and exploit misconfigurations such as unnecessary services or legacy options. | Security misconfigurations can not only expose sensitive user data, but also system details that may lead to full server compromise. |

## API ���Ǝォ�ǂ����̊m�F

�ȉ��̏ꍇ�AAPI �͐Ǝ�ł���\��������B

* �K�؂ȃZ�L�����e�B�n�[�h�j���O���A�A�v���P�[�V�����X�^�b�N�̂����镔���ł����Ă���A�������̓N���E�h�T�[�r�X�ŕs�K�؂ɐݒ肳�ꂽ�p�[�~�b�V����������ꍇ
* �ŐV�̃Z�L�����e�B�p�b�`���K�p����Ă��Ȃ��A�������̓V�X�e�����Â��B
* �s�v�ȋ@�\���L��������Ă��� (���Ƃ��΁AHTTP ���\�b�h)�B
* �g�����X�|�[�g�w�Z�L�����e�B (TLS) �������Ă���B
* �Z�L�����e�B�f�B���N�e�B�u�́A�N���C�A���g�ɑ��M����Ȃ� (���Ƃ��΁A[Security Headers][1])�B
* �N���X�I���W�����\�[�X���L (CORS) �|���V�[�������Ă���A�������͕s�K�؂ɐݒ肳��Ă���B
* �G���[���b�Z�[�W�ɃX�^�b�N�g���[�X���܂܂�Ă���A�������͑��̋@����񂪌��J����Ă���B

## �U���V�i���I��

### �V�i���I #1

�U���҂́A`.bash_history` �t�@�C�����T�[�o�̃��[�g�f�B���N�g���̉��ɂ��邱�ƂɋC�t���B�����Ă��̃t�@�C���́ADevOps �`�[���� API �ɃA�N�Z�X���邽�߂Ɏg�p����R�}���h���܂܂�Ă���B

```
$ curl -X GET 'https://api.server/endpoint/' -H 'authorization: Basic Zm9vOmJhcg=='
```

�U���҂́ADevOps �`�[���ɂ���Ă̂ݎg�p����A�h�L�������g������Ă��Ȃ� API �ŐV���ȃG���h�|�C���g�������邩������Ȃ��B

### �V�i���I #2

����̃T�[�r�X���^�[�Q�b�g�Ƃ��邽�߂ɁA�U���҂͈�ʓI�Ȍ����G���W�����g�p���āA�C���^�[�l�b�g���璼�ڃA�N�Z�X�\�ȃR���s���[�^��T���o���B�U���҂́A��ʓI�ȃf�[�x�[�X�Ǘ��V�X�e�� (DBMS) �����s���Ă���A�f�t�H���g�|�[�g�����b�X�����Ă���z�X�g�������o�����B���̃z�X�g�̓f�t�H���g�ݒ���g�p���Ă���A���̃f�t�H���g�ݒ�ł͔F�؂������ɂȂ��Ă������߁A�U���҂� PII�A�l�̚n�D�A�F�؃f�[�^���܂ސ��S���̃��R�[�h�ւ̃A�N�Z�X��������������B

### �V�i���I #3

���o�C���A�v���P�[�V�����̃g���t�B�b�N�𒲍����邱�ƂŁA�U���҂͑S�Ă� HTTP �g���t�B�b�N���Z�L���A�ȃv���g�R�� (���Ƃ��΁ATLS) �Ŏ�������Ă���킯�ł͂Ȃ����Ƃ�˂��~�߂�B�U���҂́A���ɁA�v���t�@�C���摜�̃_�E���[�h�Ɋւ��āA���̂��Ƃ������ł��邱�ƂɋC�t���BAPI �g���t�B�b�N���Z�L���A�v���g�R���Ŏ�������Ă��鎖���ɂ�������炸�A���[�U�C���^���N�V�����̓o�C�i���ł��邽�߁A�U���҂� API ���X�|���X�T�C�Y�̃p�^�[����������B�����Ă����p���āA�����_�����O���ꂽ�R���e���c (���Ƃ��΁A�v���t�B�[���摜) �ɑ΂��ă��[�U�̚n�D���g���b�L���O����B

## �΍����@

API ���C�t�T�C�N���͈ȉ����܂ނׂ��ł���B

* �K�؂Ƀ��b�N�_�E�����ꂽ���̐v�����ȒP�ȃf�v���C�ɂȂ��锽���\�ȃn�[�h�j���O�v���Z�X
* API �X�^�b�N�S�̂̐ݒ�����r���[�A�A�b�v�f�[�g���邽�߂̃^�X�N�B���r���[�ɂ́A�\���t�@�C���AAPI �R���|�[�l���g�A�N���E�h�T�[�r�X (���Ƃ��΁AS3 �o�P�b�g�̃p�[�~�b�V����) ���܂܂�Ă���ׂ��ł���B
* �S�Ă� API �C���^���N�V�������ÓI�A�Z�b�g (���Ƃ��΁A�摜) �ɃA�N�Z�X���邽�߂̃Z�L���A�ȒʐM�`���l��
* �S�Ă̊��̍\���Ɛݒ�̗L�����Ɍp���I�ɃA�N�Z�X���邽�߂̎��������ꂽ�v���Z�X


��L�ɉ����āF
* ��O�g���[�X�⑼�̗L�v�ȏ�񂪍U���҂ɕԐM����邱�Ƃ�h�����߂ɁA�K�p�ł���̂ł���΁A�G���[���X�|���X���܂ޑS�Ă� API ���X�|���X�y�C���[�h�X�L�[�}���`���Ď��s����B
* API ������� HTTP ���\�b�h�ɂ���Ă̂݃A�N�Z�X�\�ł��邱�Ƃ��m�F����B���̑S�Ă� HTTP ���\�b�h (���Ƃ��΁A`HEAD`) �͖����������ׂ��ł���B
* �u���E�U�x�[�X�̃N���C�A���g (���Ƃ��΁AWebApp �t�����g�G���h) ����̃A�N�Z�X��z�肵�Ă��� API �́A�K�؂ȃN���X�I���W�����\�[�X���L (CORS) �|���V�[����������ׂ��ł���B


## References

### OWASP

* [OWASP Secure Headers Project][1]
* [OWASP Testing Guide: Configuration Management][2]
* [OWASP Testing Guide: Testing for Error Codes][3]
* [OWASP Testing Guide: Test Cross Origin Resource Sharing][9]

### External

* [CWE-2: Environmental Security Flaws][4]
* [CWE-16: Configuration][5]
* [CWE-388: Error Handling][6]
* [Guide to General Server Security][7], NIST
* [Let’s Encrypt: a free, automated, and open Certificate Authority][8]

[1]: https://www.owasp.org/index.php/OWASP_Secure_Headers_Project
[2]: https://www.owasp.org/index.php/Testing_for_configuration_management
[3]: https://www.owasp.org/index.php/Testing_for_Error_Code_(OTG-ERR-001)
[4]: https://cwe.mitre.org/data/definitions/2.html
[5]: https://cwe.mitre.org/data/definitions/16.html
[6]: https://cwe.mitre.org/data/definitions/388.html
[7]: https://csrc.nist.gov/publications/detail/sp/800-123/final
[8]: https://letsencrypt.org/
[9]: https://www.owasp.org/index.php/Test_Cross_Origin_Resource_Sharing_(OTG-CLIENT-007)
