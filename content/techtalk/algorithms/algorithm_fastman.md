+++
date = "2016-05-10T20:52:26-05:00"
draft = true
title = "Fastman"

+++

Write a program that breaks up a string of words with no spaces into a string with appropiate spaces.

* Consider an example that's rich enough but no tedious:
 * "fastman" --> "fast man"
* Consider an empty string or null
* The words come from custom dictionary

**Solution**

```groovy
class Fastman {

	def dictionary = ["fast","fat","man","woman"]

	private String find(String string){
		String result = ''

		if(string?.length() > 0){
			for(int i=0;i<string.length()-1;i++){
				for(int j=string.length();j!=i;j--){
					if(dictionary.contains(string.substring(i,j))){
						result += "${string.substring(i,j)} "
					}
				}
			}
			result
		}
	}

	static void main(String[] args){
		Fastman fastman = new Fastman()
		String result = fastman.find('fastman')
		assert 'fast man' == result.trim()
	}

}
```

To run the project:

```bash
groovy Fastman.groovy
```

To download the code:

```bash
git clone https://github.com/josdem/algorithms-workshop.git
cd fastman-problem
```

[Return to the main article](/techtalk/algorithms)
