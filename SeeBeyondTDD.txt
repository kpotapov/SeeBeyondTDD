=Java Unit Tests in SeeBeyond's JCD with eWay=
Or how to use Java Unit Tests [http://junit.org JUnit] in use with [http://seebeyond.com SeeBeyond] Java Collaboration Definition, or yet another way to use Test Driven Development (TDD) in your software development process. (C) 2006 Constantine Potapov (Constantine (dot) Potapov at gmail (dot) c.o.m )

==Preamble.==

I'm "test infected" for about two years (it means that I'm a big fan of Test Driven Development(TDD) and agile 
technics in a software development process ), TDD allows me to make my job better, quicker,more effective and safer.

Briefly about the task that I have had.
I needed to convert a file from one data format to another.

A input OTD (java object) is loaded from a data source and converted to output OTD.

==Standard test solution==
The standard test solution based on a black box testing.
The obvious solution for organizing the testing process is to :                                                                                                  
** make a test file
** put a content of the test file into the input of a JCD
** run the whole project to rum JCD
** get a result file 
** compare a hand-made result file with  the received file, and decide if the test (was a success) passsed or not


The main advantage of this approach is simplicity, but a big disadvantage is complexity of Java Collaboration Definition development.


What JCD will do:
** gets a file 
** unmarshals the file content  to an input java object
** makes a mapping into an output java object
** marshals the output java object to a file

in the black box testing we need to (make) 
** make an additional parser for an input file
** get separate fields from a file
** make an additional parser for the output file
** get separate fields from a file
** compare every field from one file opposite the according field of the output file

in the white box testing process we can use already prepared java objects and get fields to compare directly.
it's obvious that we can skip four steps and reduce the test development cost.

==How to setup a test driven development for java collaboration definition==
For the following explanation we will use as an example the  project1 from eGate_Tutorial.pdf, I suppose it's available to all.
Go through the whole Tutorial1 project and deploy it.
copy the ''tutorial1Deployment1.ear'' file to the tmp dir
from directory c:/ican50/repository/data/files/environmentName/lhName/isName
where 
* ''environmentName'' is your enviroment name
* ''lhName'' - logical host name 
* ''isName'' - integration server name

unpack it, for example by
unpack.bat file
 mkdir tmp
 cd tmp
 jar xf ../tutorial1Deployment1.ear

This ear file contains a lot of jar files,( contains) java classes for our JCD and attendandt classes and libraries, which will be unpacked into the tmp subdirectory.


The following steps are IDE independed but will be described for the [http://www.eclipse.org Eclipse IDE] as example and can ne easy make for any other IDE.
When the SeeBeyond project deployed to a localhost, it makes a "*.ear" file, where "*" has to be replaced to projectName + deploymentName (+ sign is string concatenation )
e.g. project1Deployment1.ear

===Test Driven Development in the Eclipse IDE===

* make a new java project "TDD" ( ''projectHome'' futher in text ), the real path of the project is "C:\home\kep\projects\TDD" (for example, as you know)
* "mkdir C:\home\kep\projects\TDD\lib" ( ''LIB directory'' futher )
* copy to ''LIB directory''  from the directory where unpacked files are, from the "ear" file. all files exclude the following files
** File1_Service1.jar 
** Service1.jar 
** Service1_File2.jar 
May be not all of them were needed for java project building, but for time saving I added it all.

* copy tutorial1/Collaboration_1.java from the Service1.jar to ''projectHome''\src
( you have to see src/tutorial1/Collaboration_1.java file  )

* click the right mouse button on the project name in eclipse package explorer, choose Properties, Java Build Path, Libraries Tab and click to the "Add jars" button and add to the project library all jar from the ''project name''/lib directory 

'''Note:''' if you have copied files to ''LIB directory'' by hand, you should refresh your project (press F5 key on the TDD project name)

Here is the java collaboration source code. 
<pre>
package tutorial1;


public class Collaboration_1
{

    public com.stc.codegen.logger.Logger logger;

    public com.stc.codegen.alerter.Alerter alerter;

    public com.stc.codegen.util.CollaborationContext collabContext;

    public com.stc.codegen.util.TypeConverter typeConverter;

    public void receive( com.stc.connector.appconn.file.FileTextMessage input, com.stc.connector.appconn.file.FileApplication FileClient_1, dtd.s1input_multiple794452724.Employees s1input_multiple_Employees_1, dtd.s1output_multiple482064715.Employees s1output_multiple_Employees_1 )
        throws Throwable
    {
        s1input_multiple_Employees_1.unmarshalFromString( input.getText() );
        for (int i1 = 0; i1 < s1input_multiple_Employees_1.countEmployee(); i1 = i1 + 1) {
            s1output_multiple_Employees_1.getEmployee( i1 ).setEmpNo( s1input_multiple_Employees_1.getEmployee( i1 ).getEmployeeNumber() );
            s1output_multiple_Employees_1.getEmployee( i1 ).setFullName( s1input_multiple_Employees_1.getEmployee( i1 ).getFirstName().concat( s1input_multiple_Employees_1.getEmployee( i1 ).getLastName() ) );
        }
        FileClient_1.setText( s1output_multiple_Employees_1.marshalToString() );
        FileClient_1.write();
    }

}

</pre>

We refactor it, to facilitate testing of the collaboration code.
----
<pre>
package tutorial1;

import java.io.IOException;


public class Collaboration_1
{

    public com.stc.codegen.logger.Logger logger;

    public com.stc.codegen.alerter.Alerter alerter;

    public com.stc.codegen.util.CollaborationContext collabContext;

    public com.stc.codegen.util.TypeConverter typeConverter;

    public void receive( com.stc.connector.appconn.file.FileTextMessage input, com.stc.connector.appconn.file.FileApplication FileClient_1, dtd.s1input_multiple794452724.Employees s1input_multiple_Employees_1, dtd.s1output_multiple482064715.Employees s1output_multiple_Employees_1 )
        throws Throwable
    {
        String inputText = input.getText();
		String outputText = mapIt(s1input_multiple_Employees_1, s1output_multiple_Employees_1, inputText);
		FileClient_1.setText( outputText );
        FileClient_1.write();
    }

	public String mapIt(dtd.s1input_multiple794452724.Employees s1input_multiple_Employees_1, dtd.s1output_multiple482064715.Employees s1output_multiple_Employees_1, String inputText) throws IOException {
		s1input_multiple_Employees_1.unmarshalFromString( inputText );

        for (int i1 = 0; i1 < s1input_multiple_Employees_1.countEmployee(); i1 = i1 + 1) {
            s1output_multiple_Employees_1.getEmployee( i1 ).setEmpNo( s1input_multiple_Employees_1.getEmployee( i1 ).getEmployeeNumber() );
            s1output_multiple_Employees_1.getEmployee( i1 ).setFullName( s1input_multiple_Employees_1.getEmployee( i1 ).getFirstName().concat( s1input_multiple_Employees_1.getEmployee( i1 ).getLastName() ) );
        }

        String outputText = s1output_multiple_Employees_1.marshalToString();
		return outputText;
	}

}

</pre>
----

Make a new test case "Collaboration_1Test" in the TDD project. (may be it needs to add junut.jar to the project library, the Eclipse IDE suggest to do it by it self )

make in the root directory if the TDD project file EclipseIDEInputTDD.xml
<pre>
<?xml version="1.0" encoding="UTF-8"?>
<Employees>
  <Employee>	
	<EmployeeNumber>111</EmployeeNumber>
	<LastName>Doe</LastName>
	<FirstName>John</FirstName>
	<JobTitle>Manager</JobTitle>
	<HoursWorked>40</HoursWorked>
	<Rate>55</Rate>
  </Employee>
</Employees>
</pre>

This is a test class for the Collaboration_1 Java Collaboration Definition
<pre>
/**
 * @author Constantine Potapov (c) 2006, ( e-mail constantine.potapov@gmail.com )
 */
package tutorial1;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import junit.framework.TestCase;

public class Collaboration_1Test extends TestCase {

	/**
	 * a test method for the mapping from the Collaboration_1 (example of Java Collaboration Definition ,
	 * which translates one file format to another )
	 * @throws IOException
	 */
	public void testMapIt() throws IOException {
		// get test file content
		String inContentString = readFileAsString( "EclipseIDEInputTDD.xml" );

		// make new objects
		dtd.s1input_multiple794452724.Employees s1input_multiple_Employees_1 = new dtd.s1input_multiple794452724.Employees();
		dtd.s1output_multiple482064715.Employees s1output_multiple_Employees_1 = new dtd.s1output_multiple482064715.Employees();
		Collaboration_1 collaboration = new Collaboration_1();
		// make a mapping
		String resultOfMapping = collaboration.mapIt(s1input_multiple_Employees_1, s1output_multiple_Employees_1, inContentString);
		// test the result of mapping
		assertEquals( "111", s1output_multiple_Employees_1.getEmployee(0).getEmpNo() );
		assertEquals( "JohnDoe", s1output_multiple_Employees_1.getEmployee(0).getFullName() );
	}

	/**
	 * the method reads a String from a file
	 * @param fileName is a file name string
	 * @return  contents of a file
	 * @throws java.io.IOException
	 */
	public static String readFileAsString( String fileName ) throws java.io.IOException{
        StringBuffer stringBuffer = new StringBuffer(1024);
        BufferedReader bufferedReader = new BufferedReader(
                new FileReader(fileName));
        char[] buffer = new char[1024];
        int numRead=0;
        while((numRead=bufferedReader.read(buffer)) != -1){
            String readData = String.valueOf(buffer, 0, numRead);
            stringBuffer.append(readData);
            buffer = new char[1024];
        }
        bufferedReader.close();
        return stringBuffer.toString().replaceAll("\\r","");
    }

}

</pre>

==Disclamer==
The listed trademarks of the following companies require marking and attribution:

SeeBeyond is a registered trademark of SeeBeyond Technology Corporation in the United States and select foreign countries.

Java and all Java-based trademarks are trademarks of Sun Microsystems, Inc. in the United States, other countries, or both.

Microsoft, Windows, Windows NT, and the Windows logo are trademarks of Microsoft Corporation in the United States, other countries, or both.

Intel, Intel logo, Intel Inside, Intel Inside logo, Intel Centrino, Intel Centrino logo, Celeron, Intel Xeon, Intel SpeedStep, Itanium, and Pentium are trademarks or registered trademarks of Intel Corporation or its subsidiaries in the United States and other countries.

UNIX is a registered trademark of The Open Group in the United States and other countries.

Linux is a registered trademark of Linus Torvalds in the United States, other countries, or both.

Other company, product, or service names may be trademarks or service marks of others.
All other brands or product names are trademarks of their respective owners.


==ACKNOWLEDGMENTS==

A lot of thanks to my friend, who helped me with writing this small article.
Igor Babalich for reviewing, and Irina Fomina for english lessons.
