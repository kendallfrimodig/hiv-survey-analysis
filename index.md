---
title: "Survey Analysis of HIV Medication Preference Data Set"
author: "Kendall Frimodig"
output: github_document
---

	%macro import;
		proc import datafile="&folder\tupp2.csv"
		out=TUPP dbms=csv replace;
		getnames=yes;
		run;
	%mend import;

*********************************************************************************************
Rename variables, convert multivariable information (PID) into seperate variables
string to numeric conversions, age acalculations
*********************************************************************************************;
	%import;
DATA TUPP
	(drop= 	INTTIME
			PIDtemp
			PID
			INTEND

	 rename=(	DEM1=	sex
				DEM3=	language10
				DEM4=	education
				DEM5=	employed
				DEM6=	incomesource
				DEM7=	residence_length
				VAR56=	KPS25a
				GKH9=	GKH9a
				GKH9a=	GKH9b
				GKH9b=	GKH9c
				GKH9c=	GKH9d
				GKH9d=	GKH9e
				GKH9e=	GKH9f
				GKH9f=	GKH9g
				GKH9g=	GKH9h
				GKH9h=	GKH9i
				GKH9i=	GKH9j));

SET TUPP;

	birthdate		= PUT(dem2,8.);
	interviewdate	= COMPRESS(PUT(intdate,8.),' ');
	PIDfull			= COMPRESS(PID,,'a');
	inttime2		= COMPRESS(inttime,,'a');
	intend2			= COMPRESS(intend,,'a');
	Group_IDC 		= SUBSTR(PIDfull,1,1);
	Naive_IDC 		= SUBSTR(PIDfull,3,1);
	PIDtemp			= SUBSTR(PIDfull,4,3);
	PIDc			= COMPRESS((Group_IDC || PIDtemp),'');

	if dem2 =. then delete;

		else if dem2 > 102000 and dem2 < 120219978 then do;
			bday 	= SUBSTR(birthdate,1,2);
			bmonth  = SUBSTR(birthdate,3,2);
			byear 	= SUBSTR(birthdate,5,4);
			end;

		else if dem2<2020 then do;
			bday 	= ' ';
			bmonth 	= ' ';
			byear 	= COMPRESS(birthdate,' ');
			end;

		else if dem2=120219978 then do;
			bday	= '12';
			bmonth	= '02';
			byear	= '1978';
			end;

		else do;
			bday	= ' ';
			bmonth	= ' ';
			byear	= ' ';
			end;


	if intdate < 10000000 then do;
		intday	 = SUBSTR(interviewdate,1,1);
		intmonth = SUBSTR(interviewdate,2,2);
		intyear  = SUBSTR(interviewdate,4,4);
		end;

	else do;
		intday	 = SUBSTR(interviewdate,1,2);
		intmonth = SUBSTR(interviewdate,3,2);
		intyear  = SUBSTR(interviewdate,5,4);
		end;



	if ID >= 652 and ID <= 661 then delete;
run;


DATA TUPP
	(drop= 	PIDfull bday byear bmonth  PIDc GROUP_IDC NAIVE_IDC
			INTTIME2 INTEND2 interviewdate birthdate dem2
			intdate ID intday intmonth intyear					);
SET TUPP;

	PID 		= INPUT(PIDc,4.);
	GROUP 		= INPUT(Group_IDC,1.);
	NAIVE 		= INPUT(Naive_IDC,1.);

	INTTIME		= INPUT(INTTIME2,4.);
	INTEND 		= INPUT(INTEND2,4.);

	birthday 	= INPUT(bday,2.);
	birthmonth	= INPUT(bmonth,2.);
	birthyear	= INPUT(byear,4.);

	interviewday	= INPUT(intday,2.);
	interviewmonth	= INPUT(intmonth,2.);
	interviewyear	= INPUT(intyear,4.);

	if interviewyear = . then do;
		interviewyear = 2015;
		end;

	if interviewyear = 105 then do;
		interviewyear = 2015;
		interviewday = 8;
		interviewmonth = 9;
		end;

	if interviewmonth = 15	 then do;
		interviewmonth = 5;
		 end;

	if birthyear in(191,9200,2015,.) then delete;

	if birthyear = 1884 then do;
		birthyear=1984;
		end;
	if birthmonth = 15 then do;
		birthmonth=5;
		end;
	if birthmonth in(14,40) then do;
		birthmonth=4;
		end;
	if birthday = 61 then do;
		birthday=6;
		end;
run;


PROC SORT DATA=TUPP;
	BY PID;
run;

*********************************************************************************************
Label variables, convert -66 to 0, calculate a precise age from year, month, and day

	~~~~idea~~~~
	Idea to automate this process.. run an array of all variables, make a delimited file or
	string variable containing all of the labels, and for each iteration (i) the variable is increased
	and the SCAN(list, i) would be used to fetch the label without having to one by one
	do the GKH8= 'heard of aids'and then having to repeat all that code in the subsequent sections.
*********************************************************************************************;

DATA TUPP;
SET TUPP;

	LABEL 	GKH8	= 'Heard of Aids'
			GKH9a	= 'Abstain'
			GKH9b	= 'Condoms'
			GKH9c	= 'Shower'
			GKH9d	= 'Limit Partners'
			GKH9e	= 'No Sharing Razors'
			GKH9f	= 'Medicine from a Sangoma'
			GKH9g	= 'Faithful to Partner'
			GKH9h	= 'Sleep With a Virgin'
			GKH9i	= 'Not Engaging Sex Workers'
			GKH9j	= 'Other'
			GKH10	= 'Cure for Aids?'
			GKH11	= 'Can you Die from Aids'
			HRB12	= 'Sexually Active?'
			HRB13	= 'Virginity Age'
			HRB14	= 'Current Num Partners'
			HRB15	= 'Num Sexual Partners in Month'
			HRB16	= 'Condom used in last month'
			HRB17	= 'knows partner HIV status'
			PHR18	= 'beleive partner increases HIV risk'
			PHR19	= 'How worried about getting'
			PHR20a	= 'none'
			PHR20b	= 'currently abstaining'
			PHR20c	= 'use condoms regularly'
			PHR20d	= 'having one partner'
			PHR20e	= 'regular HIV testing'
			PHR20f	= 'knowing partner status'
			PHR20g	= 'other'
			KPS21	= 'heard of HIV prevention drugs'
			KPS22a	= 'participated in study'
			KPS22b	= 'DTHF'
			KPS22c	= 'family member'
			KPS22d	= 'spouse'
			KPS22e	= 'friends'
			KPS22f	= 'other source'
			KPS23	= 'taken part in studies'
			KPS24a	= 'vaccine study'
			KPS24b	= 'oral PREP'
			KPS24c	= 'microbicide gel study'
			KPS24d	= 'microbicide ring study'
			KPS24e	= 'not sure/dont know'
			KPS24f	= 'other'
			KPS25a	= 'for HIV negative persons'
			KPS25b	= 'for prevention'
			KPS25c	= 'not 100% effective'
			KPS25d	= 'should be used wth a condom'
			KPS25e	= 'not effective'
			KPS25f	= 'make you sick'
			KPS25g	= 'uncomfortable'
			KPS25h	= 'not sure/dont know'
			KPS25i	= 'other'
			KPS26a	= 'not 100% protective'
			KPS26b	= 'does not work'
			KPS26c	= 'works with other products'
			KPS26d	= 'works with a condom'
			KPS26e	= 'other'
			PUP27	= 'consider using these products?'
			PUP28a1	= 'partner would not allow'
			PUP28a2	= 'due to cultural practices'
			PUP28a3	= 'not comfortable'
			PUP28a4	= 'do not understand them'
			PUP28a5	= 'do not know if they work'
			PUP28a6	= 'unsure of saftey'
			PUP28a7	= 'do not like them'
			PUP28a88= 'not sure/dont know'
			PUP28a77= 'other'
			PUP28b1	= 'partner would not allow'
			PUP28b2	= 'due to cultural practices'
			PUP28b3	= 'not comfortable'
			PUP28b4	= 'do not understand them'
			PUP28b5	= 'do not know if they work'
			PUP28b6	= 'unsure of saftey'
			PUP28b7	= 'do not like them'
			PUP28b88= 'not sure/dont know'
			PUP28b77= 'other'
			PUP28c1	= 'partner would not allow'
			PUP28c2	= 'due to cultural practices'
			PUP28c3	= 'not comfortable'
			PUP28c4	= 'do not understand them'
			PUP28c5	= 'do not know if they work'
			PUP28c6	= 'unsure of saftey'
			PUP28c7	= 'do not like them'
			PUP28c88= 'not sure/dont know'
			PUP28c77= 'other'
			PUP28d1	= 'partner would not allow'
			PUP28d2	= 'due to cultural practices'
			PUP28d3	= 'not comfortable'
			PUP28d4	= 'do not understand them'
			PUP28d5	= 'do not know if they work'
			PUP28d6	= 'unsure of saftey'
			PUP28d7	= 'do not like them'
			PUP28d88= 'not sure/dont know'
			PUP28d77= 'other'
			PUP29	= 'prefered product'
			PUP30	= 'reason for preffered product'
			IPU31a	= '1st information for microbiocide use'
			IPU31b	= '2nd information for microbiocide use'
			IPU31c	= '3rd information for microbiocide use'
			IPU32	= 'would use if 50% effective?'
			IPU33a	= '1st microbiocide ring'
			IPU33b	= '2nd microbiocide ring'
			IPU33c	= '3rd microbiocide ring'
			IPU34	= 'would use ring if 50% effective?'
			IPU35a	= '1st oral PREP'
			IPU35b	= '2nd oral PREP'
			IPU35c	= '3rd oral PREP'
			IPU36	= 'would use oral prep if 50% effective'
			IPU37a	= '1st vaccine'
			IPU37b	= '2nd vaccine'
			IPU37c	= '3rd vaccine'
			IPU38	= 'would use vaccine if 50% effective'
			AAF39a	= 'easy access'
			AAF39b	= 'friend support'
			AAF39c	= 'family support'
			AAF39d	= 'partner support'
			AAF39e	= 'missing'
			AAF39f	= 'dont know, not sure'
			AAF39g	= 'other'
			AAF40a	= 'being judged about having sex'
			AAF40b	= 'being seen as having HIV'
			AAF40c	= 'forgetting'
			AAF40d	= 'no support'
			AAF40e	= 'lack of information'
			AAF40f	= 'other'
			AAF41	= 'preffer site of obtainment'
			AAF42a	= 'free'
			AAF42b	= 'acessible'
			AAF42c	= 'non-judgmental'
			AAF42d	= 'not associated with HIV+ individuals'
			AAF42e	= 'convenient'
			AAF42f	= 'anonymity'
			AAF42g	= 'special skills'
			AAF42h	= 'other'
			AAF43	= 'venue not preffered'
			AAF44a	= 'judgmental'
			AAF44b	= 'being seen as having HIV'
			AAF44c	= 'Long hours/ques'
			AAF44d	= 'rude personnel'
			AAF44e	= 'not free'
			AAF44f	= 'no privacy'
			AAF44g	= 'other'
			INTEND  = 'interview end';

	if birthmonth = . or interviewmonth= . then do;
		dem2=(interviewyear-birthyear);
		end;

		else if birthmonth > interviewmonth then do;
			dem2=(interviewyear-birthyear);
			end;

		else if  birthmonth < interviewmonth then do;
			dem2=((interviewyear-birthyear)-1);
			end;

		else if birthmonth = interviewmonth then do;
			end;


	if birthday<interviewday then do;
		dem2=((interviewyear-birthyear)-1);
		end;

		else if birthday GE interviewday then do;
			dem2=(interviewyear-birthyear);
			end;

	array a[*] _NUMERIC_;

		do i=1 TO DIM(a);

		  if a(i) = -66 then do;
		  	a(i)=0;
		  	end;
		end;
		drop i;
	run;


*********************************************************************************************************************
FORMATS
*********************************************************************************************************************;
PROC FORMAT;

		VALUE 	group
								1='Adolescents'
								2='Adults'
								3='MSM';
		VALUE 	languagetwo
								0='Other'
								1='Xhosa';

		VALUE 	languageten
								1='Xhosa'
								2='Zulu'
								3='Swati'
								4='Tshivenda'
								5='Ndebele'
								6='Tsonga'
								7='Sepedi/Northern Soto'
								8='Afrikaans/isiBhuntu'
								9='Sesotho/Southern Sotho/Sotho'
								10='Setswana/Tswana'
								11='English'
								12='Other';

		VALUE 	income  	 	1='No income'
								2='Salaries or wages'
								3='Assistance from relatives'
								4='Pensions and grants'
								77='Other'
								88='Missing';
		VALUE 	residence_length
								1='Less than 6 months'
								2='6-12 months'
								3='1-3 years'
								4='4-6 years'
								5='7 years and above';
		VALUE 	noyes
								0='No'
								1='Yes';
		VALUE 	nonotsureyes
								0='No'
								1='Not Sure'
								2='Yes';
		VALUE 	edufour
								0='No education'
								1='Primary'
								2='Lower secondary'
								3='Upper secondary'
								4='Tertiary';
		value 	eduthree
								0='Primary or lower'
								1='Lower secondary'
								2='Upper secondary';
		VALUE 	sex
								0='Male'
								1='Female';
		VALUE 	sexfive
								1='Male Youth'
								2='Female Youth'
								3='Male Adult'
								4='Female Adult'
								5='MSM Trans';

		VALUE 	prevention_knowledge
								0='Not Aware of more than 1 method'
								1='Knows 2 or more prevention methods';
		VALUE 	nosometimesyes
								0='No'
								1='Sometimes'
								2='Yes';
		VALUE 	partnerhivstatus
								0='No'
								1='Not All of Them'
								2='Yes';
		VALUE 	perceivedHIVrisk
								1='Not Worried'
								2='Not Sure'
								3='Worried a Little'
								4='Worried a Lot';
		VALUE	prefferedproduct
								1='Ring'
								2='Gel'
								3='Oral PrEP'
								4='Vaccine';
		VALUE	reasonpreffered
								1='Efficacy level'
								2='Saftey/Side Effects'
								3='Ease of Use'
								4='Familiar'
								5='Comfortable'
								6='Long Dosing Regiment'
								7='Unlikely to Forget';
		VALUE 	microgel
								1='Whether it Works'
								2='Whether its Safe'
								3='Where to find it'
								4='Whether I have to pay'
								5='Whether it will slip out'
								6='When to use it';
		VALUE 	microring
								1='How to insert it'
								2='Whether it works'
								3='Whether its safe'
								4='Where to find it'
								5='Any out of pocket cost'
								6='Whether it will slip out'
								7='Whether it is comfortable'
								8='When to use it';
		VALUE	oralprep
								1='Whether its safe'
								2='Whether it works'
								3='Where to find it'
								4='Any out of pocket cost'
								5='When to use it';
		VALUE 	vaccine
								1='Whether its safe'
								2='Whether it works'
								3='Whether its painful'
								4='Where to find it'
								5='Any out of pocket cost'
								6='When to use it';
		VALUE 	prefferedvenue
								1='Clinic'
								2='Pharmacy'
								3='Private Doctor'
								4='Hospital'
								5='Mobile Clinics'
								6='Grocery Stores'
								7='Taverns'
								8='Night Clubs'
								9='MSM Sensitized Clinic';

		Value prefferedvenuetwo
								1='clinic'
								2='pharmacy'
								3='private doctor'
								4='hospital'
								5='other';
		VALUE	notprefferedvenue
								1='Clinic'
								2='Pharmacy'
								3='Private Doctor'
								4='Hospital'
								5='Mobile Clinic'
								6='Grocery Store'
								7='Tavern'
								8='Night Club';
		VALUE study
								1='Vaccine'
								2='Oral PrEP'
								3='Microbicide Gel'
								4='Microbicide Ring';
		value sexuality
								0='heterosexual'
								1='homosexual';
		value experienced
								0='Naiive'
								1='Non-Naiive';


	run;



*********************************************************************************************************************
	~~~Bining of multi-level demographics~~~

		10 group language made into 2 categories 'xhosa' or 'other' due to most of the sample speaking 'xhosa'
		Education collapsed from grade by grade basis to 4 level
		**UPDATE** might want to change this to 2 level as theres very few no or lower primary

	~~~Question by question basis of determining what is incorecctly entered data or "missing"~~~

		This is neccessary because in the codebook for the survey, missing is entered as '88' most commonly

		however there are several cases where 		'99' is used as missing,
		or where rather than having the three level 	 77='other' 88='missing' 99='don't know/not sure'
		there is just one value such as			 88='missing/other/don't know
		or there isn't a code for missing at all....

		Frequencies were ran for each question and with consideration of the sample
		size, coding per the survey guide, and what the proportions indicated, educated guesses were made for
		what represented missing data for each question, or if 'don't know/not sure' was an important response
		to be kept depending on the nature of the question, and if it would logically fit somewhere on a spectrum
		of understanding with ordinal style questions such as level of worry regarding HIV infection


*********************************************************************************************************************;

DATA TUPP	(drop= 	PUP28d77 PUP28d88 PUP28c88
					PUP28c77 PUP28b77 PUP28b88
					PUP28a77 PUP28a88 dem2		);
SET TUPP;

		age=dem2;

		if education in(1,88) 			 		then edu4 = 0;
			else if education in(7,8,9)	 		then edu4 = 1;
			else if education in(10,11,12,13) 	then edu4 = 2;
			else if education = 14 		 		then edu4 = 3;
			else edu4=.;

		if language10 = 1 				then language2 = 1;
										else language2 = 0;

		if employed in(0,88) 				then employed_YN = 0;
			else if employed = 1 			then employed_YN = 1;

		********************************************
		if just one or 0 ways to protect were checked,
		they were not knowledgable of methods of
		prevention
		*********************************************;
		prevention_score = 0;
		myth_score = 0;

			if GKH9a = 1 then prevention_score +1;
			if GKH9b = 1 then prevention_score +1;
			if GKH9c = 1 then myth_score +1;
			if GKH9d = 1 then prevention_score +1;
			if GKH9e = 1 then prevention_score +1;
			if GKH9f = 1 then myth_score +1;
			if GKH9g = 1 then prevention_score +1;
			if GKH9h = 1 then myth_score +1;
			if GKH9i = 1 then prevention_score +1;
			if GKH9j = 77 then do;
				GKH9j = 1;
				end;

		IF prevention_score in(0,1) THEN prevention_knowledge =0;
			else if prevention_score GE 2 then prevention_knowledge=1;

		IF GKH10 = 0 THEN cure_knowledgeYN=1;
			else if GKH10 in(1,99) then cure_knowledgeYN=0;
			else cure_knowledgeYN = .;
		IF GKH10 = 0 THEN cure_knowledge=2;
			else if GKH10 =99 then cure_knowledge=1;
			else if GKH10 = 1 then cure_knowledge = 0;
			else cure_knowledge=.;
		IF GKH11 =99 THEN DO;
			GKH11=0;
			end;

		IF incomesource=88 THEN DO;
			incomesource=4;
			end;
	***********************************************************************General Knowledge of HIV;

		if GKH9j=77 then do;
			GKHn10=1;
			end;

	************************************************************************Risk Behavior Assessment;

		IF HRB12=88 then delete;		*4 missing;

		if HRB13 = 201 then do;
			HRB13=20;
			end;

		if HRB13 = 211 then do;
			HRB13 = 21;
			end;

		if HRB13 in(0,88,99) then do;		*5,141,30 imputed to missing;
			HRB13=.;
			end;

		if HRB14 in(66,88,99) then do;
			HRB14=.;
			end;

		if HRB15 in(66,88,99) then do;
			HRB15=.;
			end;

			else if HRB16 = 1 then do;
				HRB16=2;
				end;
			else if HRB16 = 2 then do;
				HRB16=1;
				end;

		if HRB17 = 88 then do;
			HRB17=.;
			end;

			else if HRB17=1 then do;
				HRB17=2;
				end;
			else if HRB17=2 then do;
				HRB17=1;
				end;
	***************************************************************************Perceived Risk of HIV;


	**************************************************************
	just 99 here, which is don't know/not sure...
	So does this mean everyone answered this question?
	no option for no response

	23 missing moved to a neutral category of perceived risk!!!!
	made a small assumption that everyone was asked this question
	***************************************************************;
	if PHR19=2 then do;
		PHR19=3;
		end;

		else if PHR19=3 then do;
			PHR19=4;
			end;
		else if PHR19=99 then do;
			PHR19=2;
			end;

	if PHR20g=77 then do;
		PHR20g=1;
		end;
	********************************************************************Knowledge of Prevention Studies;

		if KPS22d=66 then do;
			KPS22d=0;
			end;
		if KPS22f=77 then do;
			KPS22f=1;
			end;
		if KPS23=1 then do;
			KPS23=2;
			end;
		else if KPS23=99 then do;
			KPS23= 1;
			end;

		if KPS24e=77 then do; 		*the 77 and 88 are flipped per the codebook here;
			KPS24e=1;
			end;
		if KPS24f=88 then do;
			KPS24f=1;
			end;
		if KPS25b = -6 then do;
			KPS25b=0;
			end;
		if KPS25f=-660 then do;
			KPS25f=0;
			end;
		if KPS25h=99 then do;
			KPS25h=1;
			end;
		if KPS25i=77 then do;
			KPS25i=1;
			end;
		if KPS26e=77 then do;
			KPS26e=1;
			end;
	****************************************************************************Possible Use of Products;
		if PUP27=1 then do;
			PUP27 = 2;
			end;
		else if PUP27=99 then do;
			PUP27 = 1;
			end;

	****************************************************************************Information Required Before Use of Products;
	****lots of no response here, not many 'other' or not sure
		so I'm putting missing for all of them;

		if IPU34 in(2,88,99) then do;
			IPU34 =.;
			end;
		if IPU38 in(6,88) then do;
			IPU36 =.;
			end;
	*****************************************************************************Accessibility and Feasibility;

		if AAF39f=77 then do;
			AAF39f=1;
			end;
		if AAF39g=99 then do;
			AAF39g=1;
			end;
		if AAF40b=66 then delete;
		if AAF44=-6 then do;
			AAF44=0;
			end;

run;

DATA TUPP
		(keep = var1--var133 dem1--dem10 education employed language10
				myth_score prevention_knowledge prevention_score
				cure_knowledge cure_knowledgeYN INTTIME INTEND INTSTAFF		);
SET TUPP;

		dem1	=	sex;
		dem2	=	age;
		dem3	=	language2;
		dem4	=	edu4;
		dem5	=	employed_YN;
		dem6	=	incomesource;
		dem7	=	residence_length;
		dem8	=	pid;
		dem9	=	group;
		dem10	=	naive;

		var1 	= 	GKH8;
		var2 	= 	GKH9a;
		var3 	= 	GKH9b;
		var4 	= 	GKH9c;
		var5 	= 	GKH9d;
		var6 	= 	GKH9e;
		var7 	= 	GKH9f;
		var8 	= 	GKH9g;
		var9 	= 	GKH9h;
		var10 	= 	GKH9i;
		var11	= 	GKH9j;
		var12 	= 	GKH10;
		var13 	= 	GKH11;
		var14 	= 	HRB12;
		var15 	= 	HRB13;
		var16 	= 	HRB14;
		var17 	= 	HRB15;
		var18 	= 	HRB16;
		var19 	= 	HRB17;
	**************************************************************************PHR;
		var20 	= 	PHR18;
		var21 	= 	PHR19;

		var22 	= 	PHR20a;
		var23 	= 	PHR20b;
		var24 	= 	PHR20c;
		var25 	= 	PHR20d;
		var26 	= 	PHR20e;
		var27 	= 	PHR20f;
		var28 	= 	PHR20g;
	**************************************************************************KPS;
		var29 	= 	KPS21;

		var30 	= 	KPS22a;
		var31 	= 	KPS22b;
		var32 	= 	KPS22c;
		var33 	= 	KPS22d;
		var34 	= 	KPS22e;
		var35 	= 	KPS22f;

		var36 	= 	KPS23;

		var37 	= 	KPS24a;
		var38 	= 	KPS24b;
		var39 	= 	KPS24c;
		var40 	= 	KPS24d;
		var41 	= 	KPS24e;
		var42 	= 	KPS24f;

		var43 	= 	KPS25a;
		var44 	= 	KPS25b;
		var45 	= 	KPS25c;
		var46 	= 	KPS25d;
		var47 	= 	KPS25e;
		var48 	= 	KPS25f;
		var49 	= 	KPS25g;
		var50 	= 	KPS25h;
		var51 	= 	KPS25i;

		var52 	= 	KPS26a;
		var53 	= 	KPS26b;
		var54 	= 	KPS26c;
		var55 	= 	KPS26d;
		var56 	= 	KPS26e;
	****************************************************************************PUP;
		var57 	= 	PUP27;

		var58 	= 	PUP28a1;
		var59 	= 	PUP28a2;
		var60 	= 	PUP28a3;
		var61 	= 	PUP28a4;
		var62 	= 	PUP28a5;
		var63 	= 	PUP28a6;
		var64 	= 	PUP28a7;

		var65 	= 	PUP28b1;
		var66 	= 	PUP28b2;
		var67 	= 	PUP28b3;
		var68 	= 	PUP28b4;
		var69 	= 	PUP28b5;
		var70 	= 	PUP28b6;
		var71 	= 	PUP28b7;

		var72 	= 	PUP28c1;
		var73 	= 	PUP28c2;
		var74 	= 	PUP28c3;
		var75 	= 	PUP28c4;
		var76 	= 	PUP28c5;
		var77 	= 	PUP28c6;
		var78 	= 	PUP28c7;

		var79 	= 	PUP28d1;
		var80 	= 	PUP28d2;
		var81 	= 	PUP28d3;
		var82 	= 	PUP28d4;
		var83 	= 	PUP28d5;
		var84 	= 	PUP28d6;
		var85 	= 	PUP28d7;

		var86 	= 	PUP29;
		var87 	= 	PUP30;
	******************************************************************************IPU;
		var88 	= 	IPU31a;
		var89 	= 	IPU31b;
		var90 	= 	IPU31c;

		var91 	= 	IPU32;

		var92 	= 	IPU33a;
		var93 	= 	IPU33b;
		var94 	= 	IPU33c;

		var95 	= 	IPU34;

		var96 	= 	IPU35a;
		var97 	= 	IPU35b;
		var98 	= 	IPU35c;

		var99	= 	IPU36;

		var100 	= 	IPU37a;
		var101 	= 	IPU37b;
		var102 	= 	IPU37c;

		var103 	= 	IPU38;
		*************************************************************************AAF;
		var104 = 	AAF39a;
		var105 = 	AAF39b;
		var106 = 	AAF39c;
		var107 = 	AAF39d;
		var108 = 	AAF39e;
		var109 = 	AAF39f;
		var110 = 	AAF39g;

		var111 = 	AAF40a;
		var112 = 	AAF40b;
		var113 = 	AAF40c;
		var114 = 	AAF40d;
		var115 = 	AAF40e;
		var116 = 	AAF40f;

		var117 = 	AAF41;

		var118 = 	AAF42a;
		var119 = 	AAF42b;
		var120 = 	AAF42c;
		var121 = 	AAF42d;
		var122 = 	AAF42e;
		var123 = 	AAF42f;
		var124 = 	AAF42g;
		var125 = 	AAF42h;

		var126 = 	AAF43;

		var127 = 	AAF44a;
		var128 = 	AAF44b;
		var129 = 	AAF44c;
		var130 = 	AAF44d;
		var131 = 	AAF44e;
		var132 = 	AAF44f;
		var133 = 	AAF44g;

	********************************************************************************LABELS;
	LABEL
		var1		= 'Heard of Aids'
		var2		= 'Abstain'
		var3		= 'Condoms'
		var4		= 'Shower'
		var5		= 'Limit Partners'
		var6		= 'No Sharing Razors'
		var7		= 'Medicine from a Sangoma'
		var8		= 'Faithful to Partner'
		var9		= 'Sleep With a Virgin'
		var10		= 'Not Engaging Sex Workers'
		var11		= 'Other'
		var12		= 'Cure for Aids?'
		var13		= 'Can you Die from Aids'
		var14		= 'Sexually Active?'
		var15		= 'Virginity Age'
		var16		= 'Current Num Partners'
		var17		= 'Num Sexual Partners in Month'
		var18		= 'Condom used in last month'
		var19		= 'knows partner HIV status'
		var20		= 'beleive partner increases HIV risk'
		var21		= 'How worried about getting'
		var22		= 'none'
		var23		= 'currently abstaining'
		var24		= 'use condoms regularly'
		var25		= 'having one partner'
		var26		= 'regular HIV testing'
		var27		= 'knowing partner status'
		var28		= 'other'
		var29		= 'heard of HIV prevention drugs'
		var30		= 'participated in study'
		var31		= 'DTHF'
		var32		= 'family member'
		var33		= 'spouse'
		var34		= 'friends'
		var35		= 'other source'
		var36		= 'taken part in studies'
		var37		= 'vaccine study'
		var38		= 'oral PREP'
		var39		= 'microbicide gel study'
		var40		= 'microbicide ring study'
		var41		= 'not sure/dont know'
		var42		= 'other'
		var43		= 'for HIV negative persons'
		var44		= 'for prevention'
		var45		= 'not 100% effective'
		var46		= 'should be used wth a condom'
		var47		= 'not effective'
		var48		= 'make you sick'
		var49		= 'uncomfortable'
		var50		= 'not sure/dont know'
		var51		= 'other'
		var52		= 'not 100% protective'
		var53		= 'does not work'
		var54		= 'works with other products'
		var55		= 'works with a condom'
		var56		= 'other'

		var57		= 'consider using these products?'
		var58		= 'partner would not allow'
		var59		= 'due to cultural practices'
		var60		= 'not comfortable'
		var61		= 'do not understand them'
		var62		= 'do not know if they work'
		var63		= 'unsure of saftey'
		var64		= 'do not like them'

		var65		= 'partner would not allow'
		var66		= 'due to cultural practices'
		var67		= 'not comfortable'
		var68		= 'do not understand them'
		var69		= 'do not know if they work'
		var70		= 'unsure of saftey'
		var71		= 'do not like them'

		var72		= 'partner would not allow'
		var73		= 'due to cultural practices'
		var74		= 'not comfortable'
		var75		= 'do not understand them'
		var76		= 'do not know if they work'
		var77		= 'unsure of saftey'
		var78		= 'do not like them'

		var79		= 'partner would not allow'
		var80		= 'due to cultural practices'
		var81		= 'not comfortable'
		var82		= 'do not understand them'
		var83		= 'do not know if they work'
		var84		= 'unsure of saftey'
		var85		= 'do not like them'

		var86 		= 'prefered product'
		var87 		= 'reason for preffered product'
		var88 		= '1st information for microbiocide gel use'
		var89 		= '2nd information for microbiocide gel use'
		var90 		= '3rd information for microbiocide gel use'
		var91 		= 'would use microbicide gel if 50% effective?'
		var92 		= '1st microbiocide ring'
		var93 		= '2nd microbiocide ring'
		var94 		= '3rd microbiocide ring'
		var95 		= 'would use microbicide ring if 50% effective?'
		var96 		= '1st oral PREP'
		var97 		= '2nd oral PREP'
		var98 		= '3rd oral PREP'
		var99 		= 'would use oral prep if 50% effective'
		var100 		= '1st vaccine'
		var101 		= '2nd vaccine'
		var102 		= '3rd vaccine'
		var103 		= 'would use vaccine if 50% effective'
		var104 		= 'easy access'
		var105 		= 'friend support'
		var106 		= 'family support'
		var107		= 'partner support'
		var108 		= 'missing'
		var109 		= 'dont know, not sure'
		var110 		= 'other'
		var111 		= 'being judged about having sex'
		var112 		= 'being seen as having HIV'
		var113 		= 'forgetting'
		var114 		= 'no support'
		var115 		= 'lack of information'
		var116 		= 'other'
		var117 		= 'preffer site of obtainment'
		var118 		= 'free'
		var119 		= 'acessible'
		var120 		= 'non-judgmental'
		var121 		= 'not associated with HIV+ individuals'
		var122 		= 'convenient'
		var123 		= 'anonymity'
		var124 		= 'special skills'
		var125 		= 'other'
		var126 		= 'venue not preffered'
		var127 		= 'judgmental'
		var128 		= 'being seen as having HIV'
		var129 		= 'Long hours/ques'
		var130 		= 'rude personnel'
		var131 		= 'not free'
		var132 		= 'no privacy'
		var133 		= 'other'

		dem1 		= 'Sex'
		dem2 		= 'Age'
		dem3 		= 'Language 2'
		dem4		= 'Education 4'
		dem5 		= 'Employed'
		dem6		= 'Source of Income'
		dem7 		= 'Years in Community'
		dem8 		= 'PID'
		dem9 		= 'Group'
		dem10 		= 'Been in a Study Before'

		prevention_knowledge 		= 'Knows 1 or more Prevention Methods'
		Prevention_score			= 'Number of prevention methods known'
		cure_knowledgeYN 			= 'Aware there is not a cure for AIDS';

		IF dem10 in(2,.) THEN DELETE;

		FORMAT 		dem1	sex.
					dem3	languagetwo.
					dem4 	edufour.
					dem5	noyes.
					dem6	income.
					dem7	residence_length.
					dem9 	group.
					dem10	noyes.
					var1	noyes.
					var12	noyes.
					var13	noyes.
					var14	noyes.
					var18	nosometimesyes.
					var19	partnerhivstatus.
					var21	perceivedHIVrisk.
					var29	noyes.
					var30	noyes.
					var36	nonotsureyes.
					var57	nonotsureyes.
					var86	prefferedproduct.
					var87	reasonpreffered.
					var88	microgel.
					var89 	microgel.
					var90	microgel.
					var91	noyes.
					var92 	microring.
					var93	microring.
					var94	microring.
					var95	noyes.
					var96	oralprep.
					var97	oralprep.
					var98	oralprep.
					var99 	noyes.
					var100	vaccine.
					var101	vaccine.
					var102	vaccine.
					var103	noyes.
					var117	prefferedvenue.
					var126	notprefferedvenue.

					language10				languageten.
					intdate					DDMMYY.
					prevention_knowledge 	noyes.
					cure_knowlegeYN 		noyes.;

run;




DATA TUPP;
SET TUPP;

	IF var116 = 77 then do;
		var116 = 1;
		end;
	IF var125 = 77 then do;
		var125 = 1;
		end;
	IF var133 = 77 then do;
		var133 = 1;
		end;

	array a[*] _numeric_;
		do i=1 to dim(a);
			if a(i) in(-6,66,77,88,99) then do;
				a(i) = .;
				end;
			end;
	drop i;

	if var103 = 6 then do;
		var103=0;
		end;
RUN;






***********************************************************FREQS ARRAY****************************************************;

	%MACRO freq(demnum,varnum);

		proc freq data=&dataset;
		tables var&varnum*dem&demnum /NOCUM NOPERCENT NOROW NOFREQ;
		run;

	%MEND;



**Allows for the specification of a specific range of variable numbers,
all of which will be cross tabbed against all demographic variables...
if new demographic variables are added and exceed 10 then edit the dimensions
within this macro;

	%MACRO allfreq(dataset,dim2min, dim2max);

			DATA &dataset;
			SET  &dataset;
			%let varrange =	%sysevalf((&dim2max-&dim2min)+1);

			ARRAY dem[4];
			ARRAY var[&varrange] var&dim2min--var&dim2max;

			%do j=1 %to 4;

				%do i = &dim2min %to &dim2max;

					%freq(&j,&i);
				%end;
			%end;
		run;
	%MEND;




DATA 	sexually_active;
SET 	tupp;

	if var14 in(.,0) then delete;

	if var15<8 then do;
		var15=8;
		end;
RUN;



DATA 	has_current_partner;
SET 	sexually_active;

	if var16 in(0,.) then delete;
run;

DATA 	has_current_sex;
SET 	has_current_partner;

	if var17 in(0,.) 	then delete;
	if var18 = . 		then delete;

	if var19 = . then do;
		var19=0;
		end;

	if var20 = . then do;
		var20 = 0;
		end;
run;


data tupp;
set  tupp;

if var37 = 1 then study = 1;
if var38 = 1 then study = 2;
if var39 = 1 then study = 3;
if var40 = 1 then study = 4;

if dem1 in(1,3) then sex = 0;
else if dem1= 2 then sex = 1;

if dem1 = 3 then sexuality = 1;
else sexuality = 0;


format 		study 		study.
			sex 		sex.
			sexuality 	sexuality.
			dem10 		experienced.;
label	  	study = 'Type of Study Experienced in';

run;




data has_current_sex;
set has_current_sex;

if dem1=5 and var15 in(8,9) then do;
	var15=.;
	end;
run;

data tupp;
set  tupp;

if 	sexuality = . then delete;
	if sex = . then delete;
	if dem3= . then delete;
	if dem4= . then delete;
	if dem5= . then delete;
	if dem6 = . then delete;
	if dem7 = . then delete;
	if dem10 = . then delete;
	if var117 = . then delete;

	if var117=1 then venue = 1;
	if var117=2 then venue = 2;
	if var117=3 then venue = 3;
	if var117=4 then venue = 4;
	if var117 in(5,6,7,8,9) then venue = 5;

	if dem4 in(0,1) then edu3 = 0;
	if dem4 = 2 	then edu3 = 1;
	if dem4 in(3,4) then edu3 = 2;

	format  venue  prefferedvenueTWO.
			edu3 eduthree.;



run;




ods output oddsratios=nominalOR;
proc logistic data=tupp;

	class   venue		(ref="clinic")
			sexuality 	(ref="heterosexual")
			sex  		(ref="Male")
			dem3 		(ref="Xhosa")
			edu3 		(ref="Primary or lower")
			dem5 		(ref="No")
			dem6 		(ref="No income")
			dem10 		(ref="Naiive")

	/ param=ref;

	model 	venue =  sexuality sex dem2 dem3 edu3 dem5 dem6 dem10
	/
	link=glogit
	expb
	RSQUARE;

	run;
	ods output close;



	ods output oddsratios=nominalOR;
proc logistic data=tupp;

	class   var86		(ref="Oral PrEP")
			sexuality 	(ref="heterosexual")
			sex  		(ref="Male")
			dem3 		(ref="Xhosa")
			edu3 		(ref="Primary or lower")
			dem5 		(ref="No")
			dem6 		(ref="No income")
			dem10 		(ref="Naiive")

	/ param=ref;

	model 	var86 =  sexuality sex dem2 dem3 edu3 dem5 dem6 dem10
	/
	link=glogit
	expb
	RSQUARE;

	run;
	ods output close;
