设备相关：
	接口：
		1. 超管获取设备信息
		   GET /api/minder/v1/device/admin/super

		2. 校管获取设备信息
		   GET /api/minder/v1/device/admin/school
	
		3. 超管查看某个设备的流水信息
		   GET /api/minder/v1/device/record
	
	服务：
	    1. 超管获取设备服务
	    2. 超管查看某个设备的流水信息服务
	    3. 校管获取设备信息
	    4. 保存设备的流水信息服务


1. 超管通过账号查询用户和设备信息
    （1）通过账号使用userApiService的getUserDtoByUsername方法查询UserDto
    （2）通过UserDto中的unitId再通过UnitApiService的queryById方法去查询组织信息
    （3）通过uid去查询用户绑定的设备信息
    最终获得username，realname，unitName，model, systemVersion, power, sn, mac
    注意：查询的是t_student_bind中login_state = 1并且last_login = 1的学生


2. 超管普通查询用户和设备信息
    （1）数据库分页查询用户与设备信息
    （2）根据用户id结合查询用户信息List<UserDto>，拿到unitId的集合，再使用UnitApiService的queryByIdList查询组织信息


3. 超管查设备的流水
    （1）通过设备id查询设备与学生的绑定关系，拿到用户id列表
    （2）通过用户id列表通过UserClassApiService的getStudentByUserUids查询学生的信息UserStudentDto列表
    （3）UserStudentDto可以拿到班级名称（className）,再通过UserDto中的unitId再通过UnitApiService的queryByIdList查询组织信息
    最终获得loginTime, username, realname, className, unitName


4. 校管通过关键字查询用户和设备信息
    （1）后端判断关键字的类型（账号还是名字，账号是纯数字），通过StudentApiService的queryByCondition方法，传入schoolId和相应的关键字去查询学生用户信息
    （2）通过学生信息中的uid列表，去数据库中查询设备的绑定信息
    最终获得username，realname，model，systemVersion，power，sn
    注意：查询的是t_student_bind中login_state = 1并且last_login = 1的学生


5. 校管普通查询用户和设备信息
    （1）传入schoolId去数据库查询用户和设备绑定信息


