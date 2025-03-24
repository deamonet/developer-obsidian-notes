## 演示示例

这是输入需要解析xml文件：

<?xml version="1.0"?>
<class>
   <student rollno="393">
      <firstname>dinkar</firstname>
      <lastname>kad</lastname>
      <nickname>dinkar</nickname>
      <marks>85</marks>
   </student>
   <student rollno="493">
      <firstname>Vaneet</firstname>
      <lastname>Gupta</lastname>
      <nickname>vinni</nickname>
      <marks>95</marks>
   </student>
   <student rollno="593">
      <firstname>jasvir</firstname>
      <lastname>singn</lastname>
      <nickname>jazz</nickname>
      <marks>90</marks>
   </student>
</class>

**演示示例：**

_DOM4JQueryDemo.java_

package com.yiibai.xml;

import java.io.File;
import java.util.List;

import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.Node;
import org.dom4j.io.SAXReader;

public class DOM4JQueryDemo {
   public static void main(String[] args) {
      try {
         File inputFile = new File("input.txt");
         SAXReader reader = new SAXReader();
         Document document = reader.read( inputFile );

         System.out.println("Root element :" 
            + document.getRootElement().getName());

         Element classElement = document.getRootElement();

         List<Node> nodes = document.selectNodes("/class/student[@rollno='493']" );
         System.out.println("----------------------------");
         for (Node node : nodes) {
            System.out.println("\nCurrent Element :" 
               + node.getName());
            System.out.println("Student roll no : " 
               + node.valueOf("@rollno") );
            System.out.println("First Name : " + node.selectSingleNode("firstname").getText());
            System.out.println("Last Name : " + node.selectSingleNode("lastname").getText());
            System.out.println("First Name : " + node.selectSingleNode("nickname").getText());
            System.out.println("Marks : " + node.selectSingleNode("marks").getText());
         }
      } catch (DocumentException e) {
         e.printStackTrace();
      }
   }
}

这将产生以下结果:

Root element :class
----------------------------
Current Element :student
Student roll no : 493
First Name : Vaneet
Last Name : Gupta
First Name : vinni
Marks : 95

//更多请阅读：https://www.yiibai.com/java_xml/java_dom4j_query_document.html