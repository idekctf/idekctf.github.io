# algo/Class meets

> Summary: 2 students switched between in person and virtual school after `i` days of in person and `v` days of virtual, both starting in person first. Assuming each month is 30 days with 12 months, how many times do they "meet" (both being in person or both being virtual) given a starting and ending date?

> Solution: Create 3 arrays of whether or not there is school, the schedule of student1 (switching between in person and virtual), and the schedule of student2 (switching between in person and virtual). Parse through all of them at the same time and if they meet on a school day and both having the same type of school, increase a counter by one, otherwise, keep parsing.

Code:
```
def calendar(m1, d1, m2, d2, i1, i2, v1, v2):
    td1 = m1*30+d1
    td2 = m2*30+d2
```
> td1 is the total number of days from M0, D0 (starting date).
> td2 is the total number of days from M0, D0 (ending date).
```
    student1 = []
    student2 = []
    calendar_list = []
    for x in range(52):
        for z in range(5):
            calendar_list.append('s')
        for z in range(2):
            calendar_list.append('n')
```
> calendar_list is the total calendar year starting from M0, D0 switching between 5 days of `s`chool (weekdays) and 2 days of `n`o school (weekends).
```
    for before in range(td1):
        calendar_list[before] = 'n'
    for after in range(364-td2):
        calendar_list[-after] = 'n'
```
> Fill the days before the starting date and after the ending date with `n` because we don't want to count those anyways.
```
    for x in range(360//(i1+v1)+1):
        for z in range(i1):
            student1.append('i')
        for z in range(v1):
            student1.append('v')
    for x in range(360//(i2+v2)+1):
        for z in range(i2):
            student2.append('i')
        for z in range(v2):
            student2.append('v')
```
> student1 and student2 are lists for the alternating for in person and virtual for student1 and student2, respectively.
> Side note: These lists can be any length longer than calendar_list or 360 days because it doesn't matter how long they are.
```
    for x in range(len(student1)-5//7):
        student1.insert(x*5+(x-1)*2,'n')
        student1.insert(x*5+(x-1)*2,'n')
    for x in range(len(student2)-5//7):
        student2.insert(x*5+(x-1)*2,'n')
        student2.insert(x*5+(x-1)*2,'n')
```
> Remember weekends? We need to add them in for each student's list cycle (there's a `(x-1)*2` in there because adding 2 `n` in there increases the total length of the list so otherwise, the indexing is messed up).
```
    counter = 0
    for x in range(len(calendar_list)):
        if calendar_list[x] == 's':
            if student1[x] == 'i' and student2[x] == 'i':
                counter += 1
            elif student1[x] == 'v' and student2[x] == 'v':
                counter += 1
	print(counter)
```
> We loop through each day in calendar_list which is 360 days (see, the student lists' length don't matter) and if there is school and student1 "meets" student2 (being in the same school type), we increase the counter by 1. Then we print the total number of meets.


> Remember calendar(m1, d1, m2, d2, i1, i2, v1, v2) and actually run it.
> `calendar(5,6,8,3,3,4,3,6)` outputs `30`

> This is a function so you could automate it and plug it into pwntools if you so desire.

