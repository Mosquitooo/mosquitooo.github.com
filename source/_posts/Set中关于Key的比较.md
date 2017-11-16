---
title: Set中的Key
---

尘归尘, 土归土. (Your elements return to nature.)
<!-- more -->

游戏项目中遇到一个比较复杂的配置，是关于宠物价格的，发现前人用了一种很犀利的用法，记录一下。

需求： 根据宠物属性查找对应的价格

配置如下：

	等级  品质  资质下限  资质上限   价格
	10    1     2        3         10
	10    1     4        5         11
	10    2     4        4         12
	10    1     3        3         12

根据宠物等级、品质、资质来确定宠物的价格。

	// 宠物配置定义
	typedef struct Pet
	{
		int nLevel;
		int nColor;
		int nMinTel;
		int nMaxTel;
		int nPrice;

		// 重载小于符号, 给第二种方法
		friend bool operator < (const MyStruct& a, const MyStruct& b)
		{
			//等级
			if(a.nLevel < b.nLevel)
				return true;
			else if(a.nLevel > b.nLevel)
				return false;
			
			//资质
			if(a.nColor < b.nColor)
				return true;
			else if(a.nColor > b.nColor)
				return false;

			//资质有重叠则视为相等
			if(a.nMaxTel < b.nMinTel && a.nMaxTel < b.nMaxTel)
				return true;
			else if(a.nMinTel > b.nMaxTel && a.nMinTel > b.nMinTel)
				return false;
	
			return false;
		}
	}

	//假设配置表如下
	int aTestData[][5] = 
	{
		10, 1, 2, 3, 10,
		10, 1, 4, 5, 11,
		10, 2, 4, 4, 12,
		10, 1, 3, 3, 12,
	};


第一种方法： 使用multimap实现

	// 定义
	multimap<int, Pet> lPet;

	//初始化配置数据
	for(int i = 0; i < 4; i++)
	{
		Pet a;
		a.nLevel = aTestData[i][0];
		a.nColor = aTestData[i][1];
		a.nMinTel = aTestData[i][2];
		a.nMaxTel = aTestData[i][3];
		a.nPrice =  aTestData[i][4];
		lPet.insert(pair<int,Pet>(a.nLevel, a));
	}

	// 查找某个配置
	Pet tPet;
	tPet.nLevel = 10;
	tPet.nColor = 1;
	tPet.nMinTel = 3;
	tPet.nMaxTel = tPet.nMinTel; //宠物资质是一个具体的值

	typedef multimap<int, Pet>::const_iterator CTI;
	typedef pair<CTI, CTI> Range;
	Range range = lPet.equal_range(tPet.nLevel);
	
	for(CTI it = range.first; it != range.second; ++it)
	{
		if(it->second.nColor != tPet.nColor)
		{
			continue;
		}

		if(it->second.nMinTel > tPet.nMinTel || it->second.nMaxTel < tPet.nMaxTel)
		{
			continue;
		}
	
		// 找到配置
		cout <<it->second.nLevel << " " << it->second.nColor <<" "<<it->second.nPrice <<endl;
		break;
	}

这个方法的缺点：

1. 无法防止配置错误导致的问题, 在初始化配置数据时无法检查区间是否重叠，而导致同一个宠物可能会取到不同的价格


第二种方法：
	
	//定义
	set<MyStruct> sSet;
	
	//初始化配置表
	for(int i = 0; i < 4; i++)
	{
		Pet a;
		a.nLevel = aTestData[i][0];
		a.nColor = aTestData[i][1];
		a.nMinTel = aTestData[i][2];
		a.nMaxTel = aTestData[i][3];
		a.nPrice =  aTestData[i][4];

		// set中判断相等是严格弱序关系，即 !(a < b) && !(b < a) 成立则为相等
		pair<set<Pet>::iterator, bool> pair1 = sSet.insert(a);
		if (!pair1.second)
		{
			cout << "配置表中存在重复数值，请检查第" << i + 1 <<"行配置"<<endl;
		}
	}
	
	//查找某个配置
	Pet tPet;
	tPet.nLevel = 10;
	tPet.nColor = 1;
	tPet.nMinTel = 3;
	tPet.nMaxTel = tPet.nMinTel; //宠物资质是一个具体的值

	set<Pet>::iterator it1 = sSet.find(tPet);
	if (it1 != sSet.end())
	{
		cout <<it1->nLevel << " " << it1->nColor << " "<<it1->nPrice <<endl;
	}

该方法可以在策划配置有错误的时候及时检测出来, 提早发现问题！！！