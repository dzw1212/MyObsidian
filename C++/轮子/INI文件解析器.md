INI文件是一种简单的、基于文本的配置文件格式，通常用于存储应用程序的配置信息。它们以`.ini`为扩展名，广泛用于Windows平台，但也可以在其他操作系统上使用。INI文件的结构简单且易于阅读和编辑。

INI文件通常由以下几个部分组成：

1. **节（Section）**：
   - 用方括号`[]`括起来的部分，表示一个配置节。
   - 每个节可以包含多个键值对。

2. **键值对（Key-Value Pairs）**：
   - 每个键值对由一个键和一个值组成，中间用等号`=`分隔。
   - 键和值通常是字符串。

3. **注释（Comments）**：
   - 注释行以分号`;`或井号`#`开头。
   - 注释用于说明配置项的用途或其他信息。

---

以下是一个简单的INI文件示例：

```ini
; 这是一个示例INI文件

[General]
app_name = MyApp
version = 1.0

[UserSettings]
username = johndoe
email = johndoe@example.com

[Display]
resolution = 1920x1080
fullscreen = true
```

---


```cpp
//INIParser.h
#pragma once

#include <string>
#include <unordered_map>

class INIParser
{
public:
	struct SSection
	{
	public:
		SSection() = default;
		SSection(const std::string& name) :
			m_strName(name) {}
	public:
		int GetInt(const std::string& key, int defaultValue = 0) const;
		float GetFloat(const std::string& key, float defaultValue = 0.f) const;
		std::string GetString(const std::string& key, const std::string& defaultValue = "") const;

		void SetInt(const std::string& key, int value);
		void SetFloat(const std::string& key, float value);
		void SetString(const std::string& key, const std::string& value);

		std::string GetName() const;

	public:
		std::unordered_map<std::string, std::string> m_mapData;

	private:
		std::string m_strName;
	};

public:
	bool Load(const char* filepath);
	void Save(const char* filepath, bool bOverride = true);

	SSection* GetSection(const std::string& section);
	SSection* AddSection(const std::string& section);

	int GetInt(const std::string& name, const std::string& key, int defaultValue = 0) const;
	float GetFloat(const std::string& name, const std::string& key, float defaultValue = 0.f) const ;
	std::string GetString(const std::string& name, const std::string& key, const std::string& defaultValue = "") const;

	void SetInt(const std::string& name, const std::string& key, int value);
	void SetFloat(const std::string& name, const std::string& key, float value);
	void SetString(const std::string& name, const std::string& key, const std::string& value);
	
private:
	std::unordered_map<std::string, SSection> m_mapSection;
};
```

```cpp
//INIParser.cpp
#include "INIParser.h"

#include <charconv>
#include <filesystem>
#include <fstream>

int INIParser::SSection::GetInt(const std::string& key, int defaultValue) const
{
	auto it = m_mapData.find(key);
	if (it != m_mapData.end())
	{
		int value = defaultValue;
		std::from_chars(it->second.data(), it->second.data() + it->second.size(), value);
		return value;
	}
	return defaultValue;
}

float INIParser::SSection::GetFloat(const std::string& key, float defaultValue) const
{
	auto it = m_mapData.find(key);
	if (it != m_mapData.end())
	{
		float value = defaultValue;
		std::from_chars(it->second.data(), it->second.data() + it->second.size(), value);
		return value;
	}
	return defaultValue;
}

std::string INIParser::SSection::GetString(const std::string& key, const std::string& defaultValue) const
{
	auto it = m_mapData.find(key);
	if (it != m_mapData.end())
	{
		return it->second;
	}
	return defaultValue;
}

void INIParser::SSection::SetInt(const std::string& key, int value)
{
	m_mapData[key] = std::to_string(value);
}

void INIParser::SSection::SetFloat(const std::string& key, float value)
{
	m_mapData[key] = std::to_string(value);
}

void INIParser::SSection::SetString(const std::string& key, const std::string& value)
{
	m_mapData[key] = value;
}

std::string INIParser::SSection::GetName() const
{
	return m_strName;
}

bool INIParser::Load(const char* filepath)
{
	m_mapSection.clear();

	if (!std::filesystem::exists(filepath))
		return false;

	std::ifstream file(filepath);
	if (!file)
		return false;

	std::string strLine;
	SSection* curSection = nullptr;

	while (std::getline(file, strLine))
	{
		if (strLine.empty())
			continue;
		else if (strLine[0] == ';' || strLine[0] == '#')
			continue;
		else if (strLine[0] == '[')
		{
			auto nameLength = strLine.find_first_of(']') - 1;
			std::string sectionName = strLine.substr(1, nameLength);
			SSection section(sectionName);
			m_mapSection[sectionName] = section;
			curSection = &m_mapSection[sectionName];
		}
		else
		{
			auto equalSign = strLine.find_first_of('=');
			std::string key = strLine.substr(0, equalSign);
			std::string value = strLine.substr(equalSign + 1);
			if (curSection)
				curSection->SetString(key, value);
		}
	}
	return true;
}

void INIParser::Save(const char* filepath, bool bOverride)
{
	if (std::filesystem::exists(filepath) && !bOverride)
		return;

	std::ofstream file(filepath, std::ios::trunc);
	if (!file)
		return;

	for (auto& section : m_mapSection)
	{
		file << std::format("[{0}]", section.first) << std::endl;
		for (auto& pair : section.second.m_mapData)
		{
			file << std::format("{0}={1}", pair.first, pair.second) << std::endl;
		}
		file << std::endl;
	}

	file.close();
}

INIParser::SSection* INIParser::GetSection(const std::string& section)
{
	auto it = m_mapSection.find(section);
	if (it != m_mapSection.end())
		return &it->second;
	return nullptr;
}

INIParser::SSection* INIParser::AddSection(const std::string& section)
{
	auto it = m_mapSection.find(section);
	if (it != m_mapSection.end())
		return nullptr;
	SSection sec(section);
	m_mapSection[section] = sec;
	return &m_mapSection[section];
}

int INIParser::GetInt(const std::string& name, const std::string& key, int defaultValue) const
{
	auto it = m_mapSection.find(name);
	if (it == m_mapSection.end())
		return defaultValue;

	return it->second.GetInt(key, defaultValue);
}

float INIParser::GetFloat(const std::string& name, const std::string& key, float defaultValue) const
{
	auto it = m_mapSection.find(name);
	if (it == m_mapSection.end())
		return defaultValue;

	return it->second.GetFloat(key, defaultValue);
}

std::string INIParser::GetString(const std::string& name, const std::string& key, const std::string& defaultValue) const
{
	auto it = m_mapSection.find(name);
	if (it == m_mapSection.end())
		return defaultValue;

	return it->second.GetString(key, defaultValue);
}

void INIParser::SetInt(const std::string& name, const std::string& key, int value)
{
	auto it = m_mapSection.find(name);
	if (it == m_mapSection.end())
		return;

	return it->second.SetInt(key, value);
}

void INIParser::SetFloat(const std::string& name, const std::string& key, float value)
{
	auto it = m_mapSection.find(name);
	if (it == m_mapSection.end())
		return;

	return it->second.SetFloat(key, value);
}

void INIParser::SetString(const std::string& name, const std::string& key, const std::string& value)
{
	auto it = m_mapSection.find(name);
	if (it == m_mapSection.end())
		return;

	return it->second.SetString(key, value);
}

```