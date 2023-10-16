# Introduction
Amateur Skeleton Hunter, abbreviated ASH, is a helper tool designed for cosy examination on skeleton formats with as less coding work as possible. It provides a text-based C-like struct definition interface using only a handful of pre-defined variables, along with other GUI-based options for how to interpret the read data, allowing to describe different formats in a concise yet flexible way. 

![](ASH_Latest.png)

# Main Features
- Support for parsing skeleton related data of all sorts of data types;
- Ability to handle variant-length data structures via the "length" and the "skip" variables coulped with optional evaluation statements;
- Support for common arithmetic, logical and bit operations on the "length" variable;
- Support for serial evaluation statements under a specified condition via the "scope" variable;
- Formatting operations for debugging raw/converted transformation info, name mapping and bone hierarchy;
- Visualization of skeletons;
- Export of the skeletons either as the FBX format, or as an exchange node dump that can be loaded by AXE.

# System Requirement
Windows XP, Windows 7, Windows 8 or Windows 10 with Microsoft Visual C++ 2015 Redistributable (x86) or newer versions installed.

# Terms and Definitions
**`Bone Attributes`**: properties that owned by a bone node, including bone index, parent index, bone name, and transformation info such as translation, rotation and scaling (collectively referred to as the TRS components), or combined transformation matrix. The combined transformation matrix has a higher priority than individual transformation components. There're also a few accessary properties that serve as a means to determine certain properties if they don't exist in an explicit form, including bone name index and bone name address for determining the bone name, bone hash for determining the bone index, and parent hash or parent name for determining the parent index.

**`Bone Object Layout`**: layout definition of a struct holding the per-bone attributes. Each attribute can only be defined no more than once in the scope of each Branch. The mandatory attributes of a bone node, including bone index, parent index, bone name and transformation info, will be using the default values if they're not defined. The premise is that the struct is not empty.

**`String Pool Bone Name Object Layout`**: layout definition of a string element in the string pool. The only allowed variables in this layout are `skip`, `length` and `boneName`. This layout is only needed if the bone names are stored together into a buffer and referenced by an string index (or implicitly the bone index), or an address to the string. If it's referenced by index, a count of the string elements and a base address of the buffer is required. If it's referenced by address, a base address can also be specified for relative addressing.

**`Sibling Objects`**: a collection of bone object layouts which each holds exclusively parts of the required per-bone attributes. Each layout definition also comes with an optional address where the array of such defined elements begins. If the address is omitted, then it's assumed to be 0 for the first sibling object, or the address where the array defined by the previous object ends, for following objects.

**`Branch`**: a max collection of bone nodes describable by a set of Sibling Objects. A branch is defined by a set of Sibling Objects, the bone count, and optionally the String Pool Bone Name Object Layout (plus associated base address and/or string count).

**`Bone Tree`**: a set of Branches, where each node in the collection of all bones owns a unique bone index. For instance, a Branch of the character main skeleton contains 100 bones whose bone indices covered the entire closed interval from 0 to 99, and a second Branch contains 10 accessory bones (such as weapons, etc.) whose bone indices covered the entire closed interval from 100 to 109. The major obstacle to use only one Branch to complete the task could be that there's additional data inserted among the Branches, or that each Branch uses a different layout definition.

**`Bone Forest`**: a collection of Bone Trees. Useful only if there exist multiple independent skeletons in the scene.

# Global Properties
`Endianness`: global endianness for how data are read. Default value is little.<br/>
`String Encoding`: character encoding of bone names. Default value is UTF8.<br/>
`Rotation Encoding`: representation form of rotation. Default value is Euler.<br/>
`Scene Unit`: FBX scene unit property for preview only. Default value is Centimeter, which is also what AXE uses.<br/>
`Coordinate Space`: reference frame of the transformation info. Default value is local space. <br/>
`Rotation Order`: order of the combining rotations along the X, Y and Z axes. Default value is XYZ. It's only necessary if gimbal lock occurred when using the default rotation order.<br/>
`Quaternion Storage`: storage order of the 4 Quaternion components. Default value is XYZW. Affects also on vec3 Quaternions omitting the 4th component. It's only served as a last resort if everything didn't work.<br/>
`Original Axis System`: axis system the format used. Target system follows the OpenGL standard, ie. Y up, Z front, and X right. So does the default value.<br/>
`Major Order`: storage order of rotation/transformation matrix. Default value is Row-major. Applies also to Quaternion/Axis Angle where it'll perform a transpose operation on the rotations, but no effects on Eluer rotations. This property is ignored if the matrices are in 3x4 form (i.e. TransformMatrix[12]) since row-major would then be the only option.<br/>
`Transformation (Inverse Transformation)`: option for inverting the transformation info, regardless of the original form it used.

Note that the above properties take affect in a global level irrelevant to the Sibling Objects/Bone Tree/Bone Forest.

# Data Interpretation/Formatting
`Hierarchy`: print a hierarchy of the converted bone tree.<br/>
`Bone Names`: print a list of names of the converted bone tree.<br/>
`Transformations`: print the transformation info of each node of the converted bone tree.<br/>
`Bone Index to Bone Name Mapping`: print the mapping of the bone nodes and their names.<br/>
`Raw Translations/Rotations/Scalings/Transformation Matrices`: corresponding raw transformation info of a loaded Branch.

If there's no Branch/Tree in the list, it'll perform an automatic promotion from the layout definition, or failed if no layout has been defined.

# Syntax of the Layout Definition Interface
Since it's a pretty common case to see variant-length or optional data elements in bone data structure it's not possible to design a pure graphical interface while keeping everything simple and flexible. Hence a more delightful solution will be using a text-based struct definition interface that shares a similar syntax to the C programming language, except that there's no semicolon at the end of each variable declaration statement.<br/>A variable declaration statement follows a simple syntax shown below:
```
Type	Variable[Size][Size] // comment here.
```
Where "Type" stands for the allowed general data type for "Variable", and "Size" in the square brackets is an optional field that specifies the 1st and the 2nd dimension sizes if the variable is an array. Only certain variables support 2 dimensions to be defined, while the rest of them allow only 1 or 0 dimension. Single line comment is allowed using the "//" identifier.

There's no limit for the number of white spaces (space or tab) placed between "Type" and "Variable", and that after the last dimension size.

The pre-defined variables are listed below:
```
Integer		length
Eval		length[operator][num]
State		scope
State		loop[num|length]
Loop		loop[num|length]
Seek		address[num|length]
Tell		address[length]
Integer		skip[num|length]
Integer		skip[num|length][num]
Integer		skip[num][num|length]
Integer		boneIndex
Integer		boneNameIndex
Integer		boneNameAddress
Integer		boneHash
Integer		parentIndex
Integer		parentHash
Eval		boneIndex[operator][num|length]
Eval		parentIndex[operator][num|length]
Eval		boneNameIndex[operator][num|length]
Eval		boneNameAddress[operator][num|length]
TChar		boneName[num|?|length]
TChar		parentName[num|?|length]
Real		Translation[3|4]
Real		Rotation[3|4|9]
Real		Scaling[3|4]
Real		TransformMatrix[12|16]
```

The "operator" refers to one of the pre-defined operators (see below). The "num" refers to a literal value which can be either a decimal integer like 1234 or a hex value with the "0x" prefix like 0x789C (or 0x789c). The variable "length" can be used as a dimension size for variables including "skip", "boneName" and "parentName". Its default value is set to 0 if it hasn't been defined in the layout, and it'll survive within the scope of the Sibling Objects list. The '?' mark is used as a notation for null-terminated string with unpredictable length. Various size candidates are separated by the '|' mark.

Custom variables can be defined as the following types:
```
Integer		CustomVariable
Real		CustomVariable
Eval		CustomVariable[operator][num|var]
Typedef		CustomVariable[Integer|Real]
```
The "var" can be any defined custom variable except for an alias type. It can also be one of the pre-defined variables including length, boneIndex, parentIndex, boneNameIndex, and boneNameAddress. By default the custom variable mode is disabled and can be activated from the "Options" menu.

The instantiated data types of each general type are listed below:
```
Integer: char|byte|short|word|int24|uint24|long|uint32
Real:    half|short|float|double
TChar:   char|byte|word
Eval:    calc|test|assert
State:   begin|end
Loop:    break
Seek:    seek|align
Tell:    tell
Typedef: typedef
```
The variable "scope" can be used to achive serial evaluation statements when a given condition is met. In that case the serial statements body must be wrapped within the "begin scope" and the "end scope" statements. The variable "loop" can be used to perform loop operations.

The supported operators for each instantiation of the general type "Eval" are listed below:
```
calc: + - * / % & | ^ << >> =
test|assert: == != > < >= <=
```
