API6:2019 - Mass Assignment (�ꊇ���蓖��)
===========================

| ���ЃG�[�W�F���g/�U���o�H | �Z�L�����e�B��̎�_ | �e�� |
| - | - | - |
| API �ˑ� : �U����Փx **2** | �����x **2** : ���o��Փx **2** | �Z�p�I�e�� **2** : �r�W�l�X�ˑ� |
| �G�N�X�v���C�g���s���ɂ́A�ʏ�A�r�W�l�X���W�b�N�A�I�u�W�F�N�g�̊֌W�AAPI �\���̗�����K�v�Ƃ��Ă���B�v���p�e�B�̖��O�ƂƂ��ɃA�v���P�[�V�����̊�{�I�Ȏ������g�p����J���Ă��邽�߁AMass Assignment �̈��p�� API ���ȒP�ł���B | ����̃t���[�����[�N�́A�J���҂ɑ΂��ăN���C�A���g����R�[�h�ϐ�������I�u�W�F�N�g�Ɏ����I�ɓ��͂��o�C���h����֐����g�p���邱�Ƃ𐄏����Ă���B�U���҂͂��̕��@��p���āA�����ĊJ���҂����J���悤�Ƃ��Ă��Ȃ��@���ȃI�u�W�F�N�g�̃v���p�e�B�̃A�b�v�f�[�g��㏑�����s�����Ƃ��ł���B | �G�N�X�v���C�g�́A�������i�A�f�[�^������A�Z�L�����e�B���J�j�Y���̃o�C�p�X�ȂǂɂȂ��鋰�ꂪ����B |


## API ���Ǝォ�ǂ����̊m�F

����̃A�v���P�[�V�����̃I�u�W�F�N�g�́A�����̃v���p�e�B���܂�ł���\��������B�����̃v���p�e�B�̂������́A�N���C�A���g�ɂ���Ē��ڃA�b�v�f�[�g�����ׂ��ł��� (���Ƃ��΁A`user.first_name` �� `user.address`)�A�������̓A�b�v�f�[�g�����ׂ��ł͂Ȃ� (���Ƃ��΁A`user.is_vip` �t���O)�B

�����̃v���p�e�B�̋@�����ƌ��J���x���̍l���Ȃ��ɁA�N���C�A���g�p�����[�^������I�u�W�F�N�g�v���p�e�B�Ɏ����ϊ�����ꍇ�AAPI �G���h�|�C���g�͐Ǝ�ł���B����ɂ���āA�A�N�Z�X����ׂ��ł͂Ȃ��I�u�W�F�N�g�v���p�e�B���U���҂��A�b�v�f�[�g�ł���悤�ɂȂ�\��������B

Examples for sensitive properties:
�@���ȃv���p�e�B�̗�F

* **�����֘A�̃v���p�e�B**: `user.is_admin` �� `user.is_vip` �́A�Ǘ��҂ɂ���Ă̂ݐݒ肳���ׂ��ł���B
* **�v���Z�X�ˑ��̃v���p�e�B**: `user.cash` �́A�x�����m�F��A�����ł̂ݐݒ肳���ׂ��ł���B
* **�����v���p�e�B**: `article.created_time` �́A�A�v���P�[�V�����ɂ���ē����ł̂ݐݒ肳���ׂ��ł���B

## �U���V�i���I��

### �V�i���I #1

���C�h�V�F�A�����O�A�v���P�[�V�����́A�v���t�@�C���Ɋւ����{����ҏW����I�v�V���������[�U�ɒ񋟂���B���̃v���Z�X�̊Ԓ��AAPI �Ăяo���́A�ȉ��̂悤�Ȑ��K�� JSON �I�u�W�F�N�g�ƂƂ��� `PUT /api/v1/users/me` �ɑ��M�����B

```json
{"user_name":"inons","age":24}
```

���N�G�X�g `GET /api/v1/users/me` �ɂ́A�ǉ��� credit_balance �v���p�e�B���܂܂��B

```json
{"user_name":"inons","age":24,"credit_balance":10}.
```

�U���҂́A�ȉ��̃y�C���[�h�ƂƂ��ɏ��߂̃��N�G�X�g���đ�����B

```json
{"user_name":"attacker","age":60,"credit_balance":99999}
```

�G���h�|�C���g�́AMass Assignment �ɑ΂��ĐƎ�ł��邽�߁A�U���҂͎x�����Ȃ��ɃN���W�b�g���󂯎��B


### �V�i���I #2

�r�f�I�V�F�A�����O�|�[�^���́A���[�U���R���e���c���A�b�v���[�h�ł���悤�ɂ��A�l�X�ȃt�H�[�}�b�g�ŃR���e���c���_�E�����[�h�ł���悤�ɂ���BAPI �𒲍������U���҂́A�G���h�|�C���g `GET /api/v1/videos/{video_id}/meta_data` ���A�r�f�I�̃v���p�e�B���܂� JSON �I�u�W�F�N�g��Ԃ����ƂɋC�t�����B���̃v���p�e�B�̈�ɁA`"mp4_conversion_params":"-v codec h264"` ������A����́A�A�v���P�[�V�������V�F���R�}���h���g�p���ăr�f�I��ϊ����Ă��邱�Ƃ��Ӗ����Ă���B

�U���҂͂܂��A�G���h�|�C���g `POST /api/v1/videos/new` �� Mass Assignment �ɑ΂��ĐƎ�ł���A�N���C�A���g���r�f�I�I�u�W�F�N�g�̔C�ӂ̃v���p�e�B��ݒ肷�邱�Ƃ��ł��邱�Ƃɂ��C�t�����B���̍U���҂́A���ӂ���l�Ƃ��� `"mp4_conversion_params":"-v codec h264 && format C:/"` ��ݒ肷��B�U���҂� MP4 �Ƃ��ăr�f�I���_�E�����[�h����ƁA���̒l�̓V�F���R�}���h�C���W�F�N�V�����������N�������낤�B

## �΍����@

* �\�ł���΁A�R�[�h�ϐ�������I�u�W�F�N�g�ɃN���C�A���g�̓��͂������I�Ƀo�C���h����悤�Ȋ֐��̎g�p�������B
* �N���C�A���g�ɂ���ăA�b�v�f�[�g�����ׂ��v���p�e�B�̂݃z���C�g���X�g�ɓo�^����
* �N���C�A���g�ɂ���ăA�N�Z�X�����ׂ��ł͂Ȃ��v���p�e�B���u���b�N���X�g�ɓo�^���邽�߂ɑg�ݍ��݋@�\���g�p����B
* �K�p�ł��̂ł���΁A���̓f�[�^�y�C���[�h�̃X�L�[�}�𖾊m�ɒ�`���Ď��s����

## References

### External

* [CWE-915: Improperly Controlled Modification of Dynamically-Determined Object Attributes][1]

[1]: https://cwe.mitre.org/data/definitions/915.html
